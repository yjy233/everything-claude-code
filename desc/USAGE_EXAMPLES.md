# Agents 使用实战示例

## 示例 1: 开发用户认证系统 (完整流程)

### 场景
需要开发一个完整的用户认证系统，包括注册、登录、邮箱验证。

### 完整流程

#### Step 1: 规划 (/plan)

```bash
/plan "开发用户认证系统，包括：
- 用户注册（邮箱+密码）
- 邮箱验证
- 用户登录
- JWT Token 管理
- 密码重置
使用 Next.js + Supabase + TypeScript"
```

**Claude (planner) 输出**:

```markdown
# Implementation Plan: User Authentication System

## Overview
Build a complete auth system with email/password registration, 
email verification, JWT login, and password reset using Next.js 
Auth Helpers and Supabase.

## Requirements
- Email validation (format + uniqueness)
- Password strength (min 8 chars, 1 uppercase, 1 number)
- Secure JWT storage (httpOnly cookies)
- Rate limiting on auth endpoints
- Email verification before login

## Architecture Changes
- New tables: users, verification_tokens, password_reset_tokens
- New API routes: /api/auth/*
- New components: LoginForm, RegisterForm, VerifyEmail
- New middleware: auth verification

## Implementation Steps

### Phase 1: Database (2 files)
1. **Create auth schema** (supabase/migrations/001_auth.sql)
   - users table with email verification status
   - verification_tokens with expiry
   - password_reset_tokens with expiry
   
2. **Set up RLS policies**
   - Users can only read their own data
   - Public registration allowed

### Phase 2: API Routes (4 files)
3. **Register endpoint** (app/api/auth/register/route.ts)
   - Validate input with Zod
   - Hash password with bcrypt
   - Create verification token
   - Send verification email

4. **Verify email endpoint** (app/api/auth/verify/route.ts)
   - Validate token
   - Update user status

5. **Login endpoint** (app/api/auth/login/route.ts)
   - Validate credentials
   - Create JWT session
   - Set httpOnly cookie

6. **Password reset endpoints** (app/api/auth/reset/*)
   - Request reset
   - Confirm reset

### Phase 3: Frontend (3 files)
7. **Register form** (components/auth/RegisterForm.tsx)
8. **Login form** (components/auth/LoginForm.tsx)
9. **Verify page** (app/verify/page.tsx)

### Phase 4: Middleware
10. **Auth middleware** (middleware.ts)
    - Protect private routes
    - Refresh JWT if needed

## Testing Strategy
- Unit: Password validation, token generation
- Integration: API endpoints, database operations
- E2E: Full registration → verification → login flow

## Risks & Mitigations
- **Risk**: Email delivery failures
  - Mitigation: Show token in dev mode, use queue in prod
- **Risk**: JWT secret exposure
  - Mitigation: Rotate secrets monthly, use env vars

## Success Criteria
- [ ] User can register with email/password
- [ ] Verification email sent and token works
- [ ] Login returns valid JWT
- [ ] Protected routes require auth
- [ ] Password reset flow works
- [ ] All tests pass with 80%+ coverage
```

---

#### Step 2: TDD 开发 (/tdd)

```bash
/tdd "实现用户注册 API，包括：
- 输入验证 (Zod schema)
- 密码哈希 (bcrypt)
- 用户创建
- 验证令牌生成
- 错误处理"
```

**Claude (tdd-guide) 执行**:

**Step 2.1: 写测试 (RED)**
```typescript
// tests/api/auth/register.test.ts
import { POST } from '@/app/api/auth/register/route'

describe('POST /api/auth/register', () => {
  it('should create user with valid data', async () => {
    const request = new Request('http://localhost/api/auth/register', {
      method: 'POST',
      body: JSON.stringify({
        email: 'test@example.com',
        password: 'Secure123'
      })
    })
    
    const response = await POST(request)
    
    expect(response.status).toBe(201)
    const data = await response.json()
    expect(data.message).toBe('Registration successful. Check your email.')
  })
  
  it('should reject invalid email', async () => {
    const request = new Request('http://localhost/api/auth/register', {
      method: 'POST',
      body: JSON.stringify({
        email: 'invalid-email',
        password: 'Secure123'
      })
    })
    
    const response = await POST(request)
    expect(response.status).toBe(400)
  })
  
  it('should reject weak password', async () => {
    const request = new Request('http://localhost/api/auth/register', {
      method: 'POST',
      body: JSON.stringify({
        email: 'test@example.com',
        password: '123'
      })
    })
    
    const response = await POST(request)
    expect(response.status).toBe(400)
  })
  
  it('should reject duplicate email', async () => {
    // First registration
    await POST(new Request('http://localhost/api/auth/register', {
      method: 'POST',
      body: JSON.stringify({
        email: 'existing@example.com',
        password: 'Secure123'
      })
    }))
    
    // Second registration with same email
    const response = await POST(new Request('http://localhost/api/auth/register', {
      method: 'POST',
      body: JSON.stringify({
        email: 'existing@example.com',
        password: 'Secure123'
      })
    }))
    
    expect(response.status).toBe(409)
  })
})
```

**运行测试**:
```bash
npm test
# ❌ Tests fail (expected - no implementation yet)
```

**Step 2.2: 实现代码 (GREEN)**
```typescript
// app/api/auth/register/route.ts
import { z } from 'zod'
import bcrypt from 'bcrypt'
import { supabase } from '@/lib/supabase'
import { generateToken } from '@/lib/tokens'

const registerSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain uppercase')
    .regex(/[0-9]/, 'Password must contain number')
})

export async function POST(request: Request) {
  try {
    const body = await request.json()
    
    // Validate input
    const result = registerSchema.safeParse(body)
    if (!result.success) {
      return Response.json(
        { error: result.error.errors[0].message },
        { status: 400 }
      )
    }
    
    const { email, password } = result.data
    
    // Check if user exists
    const { data: existingUser } = await supabase
      .from('users')
      .select('id')
      .eq('email', email)
      .single()
    
    if (existingUser) {
      return Response.json(
        { error: 'Email already registered' },
        { status: 409 }
      )
    }
    
    // Hash password
    const hashedPassword = await bcrypt.hash(password, 12)
    
    // Create user
    const { data: user, error: createError } = await supabase
      .from('users')
      .insert({
        email,
        password_hash: hashedPassword,
        email_verified: false
      })
      .select()
      .single()
    
    if (createError) {
      throw createError
    }
    
    // Generate verification token
    const token = generateToken()
    await supabase
      .from('verification_tokens')
      .insert({
        user_id: user.id,
        token,
        expires_at: new Date(Date.now() + 24 * 60 * 60 * 1000) // 24h
      })
    
    // Send verification email (async, don't wait)
    sendVerificationEmail(email, token)
    
    return Response.json(
      { message: 'Registration successful. Check your email.' },
      { status: 201 }
    )
    
  } catch (error) {
    console.error('Registration error:', error)
    return Response.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}
```

**运行测试**:
```bash
npm test
# ✅ All tests pass
```

**Step 2.3: 重构 (IMPROVE)**
```bash
# Extract validation logic
# Extract error handling
# Improve naming
```

**检查覆盖率**:
```bash
npm run test:coverage
# Coverage: 85% lines, 90% functions
```

---

#### Step 3: 代码审查 (/code-review)

```bash
/code-review app/api/auth/register/
```

**Claude (code-reviewer) 输出**:

```markdown
## Code Review Results

### app/api/auth/register/route.ts

[HIGH] Inconsistent error message style
Line 23, 30, 52: Some errors return `error`, others throw
Fix: Standardize error response format

[MEDIUM] Missing rate limiting
File: route.ts
Issue: No protection against brute force registration
Fix: Add rate limiting middleware

[HIGH] Token generation not cryptographically secure
Line 48: generateToken() needs to use crypto.randomBytes
Fix: Use `crypto.randomBytes(32).toString('hex')`

[LOW] Magic number for bcrypt rounds
Line 40: await bcrypt.hash(password, 12)
Suggestion: Extract to constant `BCRYPT_ROUNDS = 12`

### Summary
| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | 0 | pass |
| HIGH | 2 | warn |
| MEDIUM | 1 | info |
| LOW | 1 | note |

Verdict: WARNING — Address HIGH issues before merging.
```

**修复问题**:
```bash
# 修复 token 生成
# 添加 rate limiting
# 统一错误格式
```

---

#### Step 4: 安全扫描 (/security-scan)

```bash
/security-scan app/api/auth/
```

**Claude (security-reviewer) 输出**:

```markdown
## Security Scan Results

### CRITICAL
None found ✅

### HIGH
- [ ] Rate limiting missing on all auth endpoints
  Fix: Implement express-rate-limit or similar

- [ ] CORS configuration not verified
  Fix: Ensure CORS only allows trusted origins

### MEDIUM
- [ ] Error messages could be more generic
  Line 56: Internal server error (OK, but log details)

### LOW
- [ ] Consider adding HSTS header

### Dependencies
✅ npm audit clean — no vulnerabilities found

## Recommendations
1. Add rate limiting immediately
2. Set up security headers (Helmet)
3. Add request logging for security events
```

---

#### Step 5: E2E 测试 (/e2e)

```bash
/e2e "用户注册流程：注册 → 验证邮箱 → 登录"
```

**Claude (e2e-runner) 输出**:

```typescript
// tests/e2e/auth.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Authentication Flow', () => {
  test('complete registration and login flow', async ({ page }) => {
    // Step 1: Register
    await page.goto('/register')
    await page.fill('[data-testid="email"]', 'test@example.com')
    await page.fill('[data-testid="password"]', 'Secure123')
    await page.click('[data-testid="submit"]')
    
    await expect(page.locator('[data-testid="success-message"]'))
      .toContainText('Registration successful')
    
    // Step 2: Verify email (mock email for test)
    const token = await getVerificationTokenFromDB('test@example.com')
    await page.goto(`/verify?token=${token}`)
    
    await expect(page.locator('[data-testid="verify-success"]'))
      .toContainText('Email verified')
    
    // Step 3: Login
    await page.goto('/login')
    await page.fill('[data-testid="email"]', 'test@example.com')
    await page.fill('[data-testid="password"]', 'Secure123')
    await page.click('[data-testid="submit"]')
    
    await expect(page).toHaveURL('/dashboard')
  })
  
  test('should show error for invalid credentials', async ({ page }) => {
    await page.goto('/login')
    await page.fill('[data-testid="email"]', 'wrong@example.com')
    await page.fill('[data-testid="password"]', 'wrongpass')
    await page.click('[data-testid="submit"]')
    
    await expect(page.locator('[data-testid="error"]'))
      .toContainText('Invalid credentials')
  })
})
```

**运行测试**:
```bash
npx playwright test tests/e2e/auth.spec.ts
# ✅ 2 passed
```

---

#### Step 6: 更新文档 (/update-docs)

```bash
/update-docs
```

**Claude (doc-updater) 会**:
1. 更新 README.md 添加认证功能说明
2. 创建 docs/CODEMAPS/auth.md 架构图
3. 更新 API 文档
4. 添加环境变量说明

---

## 示例 2: Bug 修复 (快速流程)

### 场景
用户报告：登录时如果密码错误，服务器崩溃而不是返回 401。

### 流程

#### Step 1: TDD - 先写复现测试

```bash
/tdd "修复登录时密码错误导致服务器崩溃的 Bug"
```

**Claude 创建测试**:
```typescript
it('should return 401 for wrong password', async () => {
  // 先创建一个用户
  await createUser('test@example.com', 'CorrectPass123')
  
  // 用错误密码登录
  const request = new Request('http://localhost/api/auth/login', {
    method: 'POST',
    body: JSON.stringify({
      email: 'test@example.com',
      password: 'WrongPass123'
    })
  })
  
  const response = await POST(request)
  
  expect(response.status).toBe(401)
  expect(await response.json()).toEqual({
    error: 'Invalid credentials'
  })
})
```

**运行测试确认失败**:
```bash
npm test
# ❌ Server crashes instead of returning 401
```

#### Step 2: 修复代码

**Claude 修复问题**:
```typescript
// 修复前 (崩溃)
const isValid = await bcrypt.compare(password, user.password_hash)
if (!isValid) {
  throw new Error('Invalid password') // ❌ 未捕获的异常
}

// 修复后 (正确处理)
const isValid = await bcrypt.compare(password, user.password_hash)
if (!isValid) {
  return Response.json(
    { error: 'Invalid credentials' },
    { status: 401 }
  )
}
```

#### Step 3: 验证

```bash
npm test
# ✅ Test passes

code-review
# ✅ No issues
```

---

## 示例 3: 代码重构 (保护性流程)

### 场景
需要重构一个 200 行的 auth service，提取重复逻辑。

### 流程

#### Step 1: 规划重构

```bash
/plan "重构 auth service，将以下逻辑提取到独立模块：
- 密码验证逻辑
- Token 生成逻辑  
- 用户信息序列化
保持现有 API 不变"
```

#### Step 2: TDD 保护现有行为

```bash
/tdd "为 auth service 添加完整测试覆盖，保护现有行为"
```

**Claude 创建全面测试**:
```typescript
// 测试所有现有功能
// 覆盖所有边界情况
// Coverage: 90%+
```

#### Step 3: 重构

```bash
# 手动或让 Claude 协助重构
```

#### Step 4: 验证

```bash
npm test
# ✅ All tests still pass

code-review
# ✅ Refactoring looks good

/refactor-clean
# ✅ Removed duplicated code
```

---

## 示例 4: 多 Agent 并行 (复杂审查)

### 场景
大型 PR，需要全面审查。

### 流程

```bash
# 同时运行多个审查
/code-review & /security-scan

# 然后
/typescript-review & /python-review  # 如果是全栈项目

# 最后
/e2e "critical flows"
```

**Claude 会并行执行**:
1. 代码质量审查
2. 安全漏洞扫描
3. 语言特定检查
4. E2E 回归测试

---

## 命令速查卡

```
┌────────────────────────────────────────────────────────────┐
│                      开发流程                               │
├────────────────────────────────────────────────────────────┤
│  /plan "功能"     →  开始新功能                              │
│  /tdd "功能"      →  开发代码                                │
│  /code-review     →  提交前审查                              │
│  /build-fix       →  修复构建错误                            │
│  /security-scan   →  安全扫描                                │
│  /e2e "流程"      →  集成测试                                │
│  /update-docs     →  更新文档                                │
├────────────────────────────────────────────────────────────┤
│                      维护流程                               │
├────────────────────────────────────────────────────────────┤
│  /refactor-clean  →  清理死代码                              │
│  /learn           →  提取模式                                │
│  /evolve          →  进化技能                                │
├────────────────────────────────────────────────────────────┤
│                      多语言审查                             │
├────────────────────────────────────────────────────────────┤
│  /typescript-review [路径]                                   │
│  /python-review [路径]                                       │
│  /go-review [路径]                                           │
│  /java-review [路径]                                         │
│  /rust-review [路径]                                         │
└────────────────────────────────────────────────────────────┘
```

---

## 关键要点

1. **始终从 /plan 开始** - 清晰的方向比快速开始更重要
2. **用 /tdd 写所有代码** - 测试保护你的代码
3. **提交前 /code-review** - 防止问题进入仓库
4. **定期 /security-scan** - 安全第一
5. **Hook 自动化** - 让 Agents 在后台帮你检查

**记住**: Agents 不是替代你思考，而是放大你的能力。你仍然需要理解问题和方案，Agents 帮你执行得更好、更系统。
