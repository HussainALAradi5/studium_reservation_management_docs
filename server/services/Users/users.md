# Users services

## AuthService — `services/Users/AuthService.php`

### `register(RegisterDto $dto): User`
**Purpose:** Self-registration. **Logic:** 1) `UserService::assertUnique` on username/email/phone; 2) insert with role `customer`, bcrypt hash, active; 3) start a fresh session. **Exceptions:** 409 `DUPLICATE_VALUE` (field error).

### `login(LoginDto $dto): User`
**Purpose:** Sign in by username or email. **Logic:** 1) count failures for identifier+IP in the last 15 min; ≥ 5 → 429; 2) load user, verify active + password; on failure record an attempt and throw 401 with a generic message; 3) clear attempts, set `last_login_at`, regenerate the session id.

| Case | Result |
| --- | --- |
| Valid credentials | Session started |
| Wrong password / unknown user / inactive | 401 `UNAUTHENTICATED` "Incorrect username or password." |
| 5 failures in 15 minutes | 429 `TOO_MANY_ATTEMPTS` |

### `logout(): void`
Destroys the session and its cookie.

### `resolveSessionUser(): ?User`
**Purpose:** Called by the Kernel on every request. Returns null when there is no session user, the session was idle longer than `SESSION_IDLE_MINUTES`, or the user was deleted/deactivated (so deactivation signs users out on their next request). Refreshes the activity timestamp.

## UserService — `services/Users/UserService.php`

| Function | Purpose | Exceptions |
| --- | --- | --- |
| `list(UserQuery $query): PageResult` | Paginated, filtered users (search, role, status) | — |
| `findByCode(string $code): User` | One user | 404 |
| `listManagers(): User[]` | Active managers for stadium assignment | — |
| `create(UserFormDto $dto): User` | Admin creates any role; password required | 409 duplicate |
| `update(User $actor, string $code, UserFormDto $dto): User` | Admin edits a user; password changes only when given | 404, 409 duplicate, 409 `SELF_ACTION_FORBIDDEN`, 409 `MANAGER_HAS_STADIUMS` |
| `toggleActive(User $actor, string $code): User` | Enable / disable | same as update |
| `updateProfile(User $user, ProfileDto $dto): User` | Own profile (not role, username, status) | 409 duplicate |
| `changePassword(User $user, ChangePasswordDto $dto): void` | Verifies the current password first | 422 on `currentPassword` |
| `assertUnique(string $username, string $email, string $phone, ?int $excludeId = null): void` | Shared duplicate check among non-deleted users | 409 `DUPLICATE_VALUE` with field |
| `countByRole(): array` | Active users per role (admin dashboard) | — |

**Role-change rule (`assertRoleChangeAllowed`):** an admin cannot change their own role or deactivate themself; a manager who still manages stadiums cannot lose the manager role or be deactivated until the stadiums are reassigned.

## PermissionService — `services/Users/PermissionService.php`

| Function | Purpose |
| --- | --- |
| `can(User $user, Permission $permission): bool` | Used by `RequirePermission` middleware and the `can()` view helper |
| `granted(UserRole $role): string[]` | Admin → every permission; others → rows of `role_permissions` (cached per request) |
| `matrix(): array` | role → granted permissions, for the settings page |
| `updateRole(UserRole $role, Permission[] $permissions): void` | Replaces a role's grants in one transaction; admin → 409 `SELF_ACTION_FORBIDDEN` |
| `seedDefaults(): void` | Writes `Permission::defaultsFor()` for every role (seed) |
