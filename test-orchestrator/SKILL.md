---
name: test-orchestrator
description: "Set up testing infrastructure and strategy for a project. This skill should be used when a project is ready for testing setup, including test framework configuration, initial test scaffolding, and documentation of testing approach. Primarily a setup skill with guidance for ongoing testing."
version: 1.1.0
author: john
tags:
  - workflow
  - quality
  - testing
  - executor
allowed-tools:
  - Read
  - Grep
  - Glob
  - Write
  - Bash
---

# test-orchestrator

<hard-boundaries>
BEFORE ANY OUTPUT, I MUST VERIFY:
1. I will NOT change tech stack decisions - tech-stack-advisor already decided
2. I will NOT change deployment target - deployment-advisor already decided
3. I will NOT suggest alternative testing frameworks without user confirmation
4. I will ONLY set up testing infrastructure for the CHOSEN tech stack
5. I will USE environment registry data for containerization context (Docker is established)

MY SCOPE: Set up testing infrastructure based on LOCKED decisions from upstream skills. I am a BUILDER and EDUCATOR, not an ADVISOR for technology choices.
</hard-boundaries>

<purpose>
Analyze an existing project and set up appropriate testing infrastructure. Creates test framework configuration, initial test scaffolding, and documents testing strategy. Provides educational guidance on testing best practices for the project's tech stack.
</purpose>

<role>
BUILDER role with CONSULTANT guidance. Sets up infrastructure and provides education.
- WILL configure testing frameworks
- WILL create initial test files and scaffolding
- WILL write test strategy documentation
- WILL provide guidance on writing tests
- Will NOT write comprehensive test suites (that's ongoing development)
</role>

<output>
- Test framework configuration files
- Initial test scaffolding (example tests)
- .docs/test-strategy.md (testing approach documentation)
- Test scripts in package.json or equivalent
- Educational guidance on testing patterns
</output>

---

<workflow>

<phase id="0" name="load-environment">
<action>Load environment context for infrastructure awareness.</action>

<environment-loading>
1. Attempt to read ~/.claude/environment.json
2. If not found: Note and proceed (graceful degradation)
3. If found: Read skill_data_access["test-orchestrator"] to get allowed paths:
   - established_choices.containerization
4. Load ONLY those sections into context
5. Use this data to inform test infrastructure decisions

Key data to extract:
- established_choices.containerization: Docker is the locked choice for all VPS deployments
</environment-loading>

<when-environment-exists>
Note: Docker is the established containerization choice. Test configurations should account for Docker-based development environments (docker-compose for local dev).
</when-environment-exists>

<when-environment-missing>
Proceed without environment context. Ask about containerization if needed for test configuration.
</when-environment-missing>
</phase>

<phase id="1" name="load-handoffs">
<action>Load upstream JSON handoff documents for context.</action>

<expected-documents>
- .docs/tech-stack-decision.json (technology choices from tech-stack-advisor)
- .docs/deployment-strategy.json (deployment and storage decisions from deployment-advisor)
- .docs/project-foundation.json (project setup details from project-spinup) - optional
</expected-documents>

<load-process>
1. Scan .docs/ for expected JSON handoff documents
2. If found: Load context and summarize conversationally
3. If missing: Gather equivalent information through project analysis
4. Proceed with skill workflow regardless
</load-process>

<extract-from-tech-stack-decision>
From .docs/tech-stack-decision.json:
- decisions[].category == "testing": Get recommended testing framework
- decisions[].category == "frontend"|"backend": Get tech stack for framework matching
- rationale: Understand project context
</extract-from-tech-stack-decision>

<extract-from-deployment-strategy>
From .docs/deployment-strategy.json:
- hosting.file_storage: Determine if storage testing is needed
- hosting.type: Understand deployment environment for integration tests
- development_environment: Know where tests will run
</extract-from-deployment-strategy>

<key-principle>
This skill NEVER blocks on missing handoffs. It gathers information from project analysis if handoffs are unavailable.
</key-principle>
</phase>

<phase id="2" name="analyze-project">
<action>Scan project to understand tech stack and existing test setup.</action>

<project-analysis>
Look for and analyze:

1. Tech stack indicators:
   - package.json (Node.js/frontend framework)
   - requirements.txt / pyproject.toml (Python)
   - composer.json (PHP)
   - *.xcodeproj or Package.swift (iOS/macOS SwiftUI)
   - src-tauri/Cargo.toml (Tauri desktop)
   - claude.md (project context)

2. Existing test configuration:
   - jest.config.js, vitest.config.ts
   - pytest.ini, pyproject.toml [tool.pytest]
   - phpunit.xml
   - tests/ or __tests__/ directories

3. Framework-specific patterns:
   - Next.js: app/, pages/, components/
   - FastAPI: app/, routers/, models/
   - PHP: src/, Controllers/, Models/

4. Existing tests:
   - Count and categorize existing test files
   - Identify testing patterns in use
</project-analysis>

<supplement-from-handoffs>
If .docs/tech-stack-decision.json was loaded:
- Validate project structure matches declared tech stack
- Use testing framework recommendation from decisions array

If .docs/deployment-strategy.json was loaded:
- Check hosting.file_storage for storage testing needs
- Note development_environment for test execution context
</supplement-from-handoffs>

<check-storage-needs>
Determine if project uses object storage (from deployment-strategy.json hosting.file_storage):

1. **No storage / None** - No file uploads, skip storage testing
2. **Local VPS storage** - Files on filesystem, test file operations
3. **Backblaze B2** - S3-compatible storage, test with mocks + integration
4. **Tigris** - Fly.io native S3-compatible, test with mocks + integration

If storage is used, include:
- Storage mocking patterns in unit tests
- Storage integration test examples
- Storage testing section in test-strategy.md
</check-storage-needs>
</phase>

<phase id="3" name="determine-testing-approach">
<action>Recommend testing strategy based on project type and tech stack.</action>

<testing-layers>
| Layer | Purpose | When to Use |
|-------|---------|-------------|
| Unit Tests | Test individual functions/components in isolation | Always - foundation of testing |
| Integration Tests | Test multiple components working together | API routes, database operations |
| E2E Tests | Test complete user flows | Critical paths, happy paths |
</testing-layers>

<framework-recommendations>

<nextjs-testing>
**Recommended Stack:**
- Vitest or Jest for unit/integration tests
- React Testing Library for component tests
- Playwright or Cypress for E2E tests

**Configuration:**
- vitest.config.ts with jsdom environment
- @testing-library/react for component assertions
- MSW (Mock Service Worker) for API mocking
</nextjs-testing>

<fastapi-testing>
**Recommended Stack:**
- pytest for all test types
- pytest-asyncio for async tests
- httpx for API testing
- Factory Boy for test data

**Configuration:**
- pytest.ini or pyproject.toml [tool.pytest]
- conftest.py for fixtures
- TestClient for FastAPI endpoint testing
</fastapi-testing>

<php-testing>
**Recommended Stack:**
- PHPUnit for unit/integration tests
- Pest PHP (optional, more readable syntax)
- Laravel testing tools (if Laravel)

**Configuration:**
- phpunit.xml
- tests/Unit/, tests/Feature/ directories
</php-testing>

<node-express-testing>
**Recommended Stack:**
- Jest or Vitest for unit tests
- Supertest for API endpoint testing

**Configuration:**
- jest.config.js
- Test database configuration
</node-express-testing>

<swiftui-testing>
**Recommended Stack:**
- XCTest (built into Xcode)
- XCUITest for UI testing

**Configuration:**
- Tests live in {ProjectName}Tests/ target
- UI tests in {ProjectName}UITests/ target
- Xcode manages test configuration automatically

**What to test:**
- ViewModels with XCTest assertions
- Service layer unit tests
- UI flows with XCUITest (optional)

**Run tests:**
- Xcode: Cmd+U or Product → Test
- CLI: `xcodebuild test -scheme {SchemeName} -destination 'platform=iOS Simulator,name=iPhone 15'`

**Note:** Native iOS/macOS testing is fully handled by Xcode. This skill provides guidance but Xcode manages the test infrastructure.
</swiftui-testing>

<tauri-testing>
**Recommended Stack:**
- Frontend: Vitest or Jest (same as web projects)
- Rust backend: cargo test (built into Rust toolchain)

**Configuration:**
- Frontend tests: vitest.config.ts in src-tauri/ or project root
- Rust tests: inline in src-tauri/src/*.rs files

**What to test:**
- Frontend components with Vitest + Testing Library
- Tauri commands (Rust functions) with cargo test
- IPC between frontend and Rust with integration tests

**Run tests:**
```bash
# Frontend tests
npm test

# Rust tests
cd src-tauri && cargo test
```

**Note:** Tauri projects use standard web testing for the frontend. The Rust backend uses Rust's built-in test framework.
</tauri-testing>

</framework-recommendations>

<prompt-to-user>
Based on your {tech-stack} project, I recommend:

**Testing Framework:** {framework}
**Test Types:**
- Unit tests for {specific areas}
- Integration tests for {specific areas}
- E2E tests for {specific areas} (optional)

**Coverage Goal:** Start with critical paths, aim for 70-80% over time

Does this approach work for you? Any specific areas you want to prioritize?
</prompt-to-user>
</phase>

<phase id="4" name="configure-framework">
<action>Set up testing framework configuration files.</action>

<configuration-files>

<vitest-config>
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./tests/setup.ts'],
    include: ['**/*.{test,spec}.{js,ts,jsx,tsx}'],
    coverage: {
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'tests/'],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```
</vitest-config>

<jest-config>
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/tests/setup.js'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
  ],
  testMatch: ['**/__tests__/**/*.[jt]s?(x)', '**/?(*.)+(spec|test).[jt]s?(x)'],
}
```
</jest-config>

<pytest-config>
```toml
# pyproject.toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_functions = ["test_*"]
asyncio_mode = "auto"
addopts = "-v --tb=short"

[tool.coverage.run]
source = ["app"]
omit = ["tests/*", "alembic/*"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
]
```
</pytest-config>

<phpunit-config>
```xml
<!-- phpunit.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
    </testsuites>
    <coverage>
        <include>
            <directory suffix=".php">src</directory>
        </include>
    </coverage>
</phpunit>
```
</phpunit-config>

</configuration-files>

<package-scripts>
Add test scripts to package.json (Node.js projects):

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:watch": "vitest --watch",
    "test:ui": "vitest --ui"
  }
}
```

Or for Python (in pyproject.toml):
```toml
[tool.poetry.scripts]
test = "pytest:main"
```
</package-scripts>
</phase>

<phase id="5" name="create-scaffolding">
<action>Create initial test files demonstrating testing patterns.</action>

<test-directory-structure>
```
tests/
├── setup.ts (or setup.js, conftest.py)
├── unit/
│   └── example.test.ts
├── integration/
│   └── api.test.ts
└── e2e/
    └── (optional) flows.test.ts
```
</test-directory-structure>

<example-unit-test-react>
```typescript
// tests/unit/example.test.tsx
import { describe, it, expect } from 'vitest'
import { render, screen } from '@testing-library/react'
import { Button } from '@/components/ui/button'

describe('Button Component', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByRole('button')).toHaveTextContent('Click me')
  })

  it('handles click events', async () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick}>Click me</Button>)

    await userEvent.click(screen.getByRole('button'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('can be disabled', () => {
    render(<Button disabled>Click me</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```
</example-unit-test-react>

<example-integration-test-api>
```typescript
// tests/integration/api.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest'

describe('API Endpoints', () => {
  beforeAll(async () => {
    // Setup: start test server, seed database
  })

  afterAll(async () => {
    // Cleanup: stop server, clear database
  })

  describe('GET /api/items', () => {
    it('returns list of items', async () => {
      const response = await fetch('http://localhost:3000/api/items')
      const data = await response.json()

      expect(response.status).toBe(200)
      expect(Array.isArray(data)).toBe(true)
    })

    it('returns 401 without authentication', async () => {
      const response = await fetch('http://localhost:3000/api/protected')
      expect(response.status).toBe(401)
    })
  })
})
```
</example-integration-test-api>

<example-pytest>
```python
# tests/test_api.py
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

class TestItems:
    async def test_get_items(self, client):
        response = await client.get("/api/items")
        assert response.status_code == 200
        assert isinstance(response.json(), list)

    async def test_create_item(self, client):
        response = await client.post(
            "/api/items",
            json={"name": "Test Item", "description": "A test"}
        )
        assert response.status_code == 201
        assert response.json()["name"] == "Test Item"

    async def test_unauthorized_access(self, client):
        response = await client.get("/api/protected")
        assert response.status_code == 401
```
</example-pytest>

<setup-file>
```typescript
// tests/setup.ts
import '@testing-library/jest-dom'
import { cleanup } from '@testing-library/react'
import { afterEach } from 'vitest'

// Cleanup after each test
afterEach(() => {
  cleanup()
})

// Global mocks
vi.mock('next/navigation', () => ({
  useRouter: () => ({
    push: vi.fn(),
    replace: vi.fn(),
    prefetch: vi.fn(),
  }),
  useSearchParams: () => new URLSearchParams(),
  usePathname: () => '/',
}))
```
</setup-file>

<storage-testing-patterns>
Include these patterns when project uses object storage (B2, Tigris, or local).

<storage-mock-typescript>
```typescript
// tests/mocks/storage.ts
// Mock for S3-compatible storage (Backblaze B2, Tigris)
import { vi } from 'vitest'

export const mockS3Client = {
  send: vi.fn(),
}

export const mockUploadResponse = {
  $metadata: { httpStatusCode: 200 },
  ETag: '"mock-etag-12345"',
  Location: 'https://bucket.s3.region.backblazeb2.com/test-file.jpg',
}

export const mockGetObjectResponse = {
  $metadata: { httpStatusCode: 200 },
  Body: {
    transformToByteArray: vi.fn().mockResolvedValue(new Uint8Array([1, 2, 3])),
    transformToString: vi.fn().mockResolvedValue('file content'),
  },
  ContentType: 'image/jpeg',
  ContentLength: 1024,
}

// Reset mocks between tests
export function resetStorageMocks() {
  mockS3Client.send.mockReset()
}

// Setup common mock responses
export function setupUploadMock() {
  mockS3Client.send.mockResolvedValue(mockUploadResponse)
}

export function setupGetObjectMock() {
  mockS3Client.send.mockResolvedValue(mockGetObjectResponse)
}

export function setupStorageErrorMock(error: Error) {
  mockS3Client.send.mockRejectedValue(error)
}
```
</storage-mock-typescript>

<storage-unit-test-example>
```typescript
// tests/unit/storage.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { mockS3Client, setupUploadMock, resetStorageMocks } from '../mocks/storage'

// Mock the AWS SDK
vi.mock('@aws-sdk/client-s3', () => ({
  S3Client: vi.fn(() => mockS3Client),
  PutObjectCommand: vi.fn(),
  GetObjectCommand: vi.fn(),
  DeleteObjectCommand: vi.fn(),
}))

import { uploadFile, getFileUrl } from '@/lib/storage'

describe('Storage Service', () => {
  beforeEach(() => {
    resetStorageMocks()
  })

  describe('uploadFile', () => {
    it('uploads file and returns URL', async () => {
      setupUploadMock()
      const file = new File(['test content'], 'test.jpg', { type: 'image/jpeg' })

      const result = await uploadFile(file, 'uploads/test.jpg')

      expect(result.success).toBe(true)
      expect(result.url).toContain('test.jpg')
      expect(mockS3Client.send).toHaveBeenCalledTimes(1)
    })

    it('handles upload errors gracefully', async () => {
      mockS3Client.send.mockRejectedValue(new Error('Network error'))
      const file = new File(['test'], 'test.jpg', { type: 'image/jpeg' })

      const result = await uploadFile(file, 'uploads/test.jpg')

      expect(result.success).toBe(false)
      expect(result.error).toBeDefined()
    })

    it('validates file type before upload', async () => {
      const file = new File(['test'], 'test.exe', { type: 'application/x-executable' })

      const result = await uploadFile(file, 'uploads/test.exe')

      expect(result.success).toBe(false)
      expect(result.error).toContain('file type')
      expect(mockS3Client.send).not.toHaveBeenCalled()
    })
  })
})
```
</storage-unit-test-example>

<storage-integration-test-example>
```typescript
// tests/integration/storage.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest'
import { S3Client, PutObjectCommand, GetObjectCommand, DeleteObjectCommand } from '@aws-sdk/client-s3'

/**
 * Storage Integration Tests
 *
 * These tests verify actual storage connectivity.
 * Run against a test bucket, not production.
 *
 * Prerequisites:
 * - TEST_STORAGE_BUCKET environment variable set
 * - Storage credentials configured (B2 or Tigris)
 *
 * Run with: npm run test:integration
 */

const TEST_BUCKET = process.env.TEST_STORAGE_BUCKET
const TEST_KEY = `test-${Date.now()}.txt`

// Skip if no test bucket configured
const runIntegrationTests = !!TEST_BUCKET

describe.skipIf(!runIntegrationTests)('Storage Integration', () => {
  let s3Client: S3Client

  beforeAll(() => {
    s3Client = new S3Client({
      endpoint: process.env.B2_ENDPOINT || process.env.AWS_ENDPOINT_URL_S3,
      region: process.env.AWS_REGION || 'auto',
      credentials: {
        accessKeyId: process.env.B2_APPLICATION_KEY_ID || process.env.AWS_ACCESS_KEY_ID!,
        secretAccessKey: process.env.B2_APPLICATION_KEY || process.env.AWS_SECRET_ACCESS_KEY!,
      },
    })
  })

  afterAll(async () => {
    // Cleanup: delete test file
    try {
      await s3Client.send(new DeleteObjectCommand({
        Bucket: TEST_BUCKET,
        Key: TEST_KEY,
      }))
    } catch {
      // Ignore cleanup errors
    }
  })

  it('can upload a file', async () => {
    const response = await s3Client.send(new PutObjectCommand({
      Bucket: TEST_BUCKET,
      Key: TEST_KEY,
      Body: 'Test content for integration test',
      ContentType: 'text/plain',
    }))

    expect(response.$metadata.httpStatusCode).toBe(200)
  })

  it('can retrieve an uploaded file', async () => {
    const response = await s3Client.send(new GetObjectCommand({
      Bucket: TEST_BUCKET,
      Key: TEST_KEY,
    }))

    expect(response.$metadata.httpStatusCode).toBe(200)
    expect(response.ContentType).toBe('text/plain')
  })
})
```
</storage-integration-test-example>

<storage-pytest-example>
```python
# tests/test_storage.py
import pytest
from unittest.mock import Mock, patch, AsyncMock
import boto3
from botocore.exceptions import ClientError

class TestStorageUnit:
    """Unit tests with mocked S3 client"""

    @pytest.fixture
    def mock_s3_client(self):
        with patch('boto3.client') as mock:
            client = Mock()
            mock.return_value = client
            yield client

    def test_upload_file_success(self, mock_s3_client):
        from app.services.storage import upload_file

        mock_s3_client.put_object.return_value = {
            'ETag': '"mock-etag"',
            'ResponseMetadata': {'HTTPStatusCode': 200}
        }

        result = upload_file(b'test content', 'test.jpg', 'image/jpeg')

        assert result['success'] is True
        mock_s3_client.put_object.assert_called_once()

    def test_upload_file_error(self, mock_s3_client):
        from app.services.storage import upload_file

        mock_s3_client.put_object.side_effect = ClientError(
            {'Error': {'Code': '500', 'Message': 'Internal Error'}},
            'PutObject'
        )

        result = upload_file(b'test content', 'test.jpg', 'image/jpeg')

        assert result['success'] is False
        assert 'error' in result


@pytest.mark.integration
class TestStorageIntegration:
    """Integration tests against real storage (test bucket)"""

    @pytest.fixture
    def s3_client(self):
        import os
        if not os.getenv('TEST_STORAGE_BUCKET'):
            pytest.skip('TEST_STORAGE_BUCKET not configured')

        return boto3.client(
            's3',
            endpoint_url=os.getenv('B2_ENDPOINT'),
            aws_access_key_id=os.getenv('B2_APPLICATION_KEY_ID'),
            aws_secret_access_key=os.getenv('B2_APPLICATION_KEY'),
        )

    def test_upload_and_retrieve(self, s3_client):
        import os
        bucket = os.getenv('TEST_STORAGE_BUCKET')
        key = f'test-{pytest.importorskip("time").time()}.txt'

        # Upload
        s3_client.put_object(
            Bucket=bucket,
            Key=key,
            Body=b'Integration test content',
            ContentType='text/plain'
        )

        # Retrieve
        response = s3_client.get_object(Bucket=bucket, Key=key)
        content = response['Body'].read()

        assert content == b'Integration test content'

        # Cleanup
        s3_client.delete_object(Bucket=bucket, Key=key)
```
</storage-pytest-example>

<local-storage-test-example>
```typescript
// tests/unit/local-storage.test.ts
// For projects using local VPS filesystem storage
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest'
import fs from 'fs/promises'
import path from 'path'

vi.mock('fs/promises')

describe('Local File Storage', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('saves file to upload directory', async () => {
    vi.mocked(fs.writeFile).mockResolvedValue()
    vi.mocked(fs.mkdir).mockResolvedValue(undefined)

    const { saveFile } = await import('@/lib/local-storage')
    const result = await saveFile(Buffer.from('test'), 'uploads/test.jpg')

    expect(fs.writeFile).toHaveBeenCalled()
    expect(result.path).toContain('test.jpg')
  })

  it('creates directory if not exists', async () => {
    vi.mocked(fs.mkdir).mockResolvedValue(undefined)
    vi.mocked(fs.writeFile).mockResolvedValue()

    const { saveFile } = await import('@/lib/local-storage')
    await saveFile(Buffer.from('test'), 'uploads/new-dir/test.jpg')

    expect(fs.mkdir).toHaveBeenCalledWith(
      expect.stringContaining('uploads/new-dir'),
      expect.objectContaining({ recursive: true })
    )
  })
})
```
</local-storage-test-example>
</storage-testing-patterns>
</phase>

<phase id="6" name="create-strategy-doc">
<action>Create .docs/test-strategy.md documenting the testing approach.</action>

<strategy-template>
```markdown
# Test Strategy

**Project:** {project_name}
**Created:** {date}
**Framework:** {test_framework}

## Testing Philosophy

This project follows a testing pyramid approach:
- Many unit tests (fast, isolated)
- Fewer integration tests (verify component interaction)
- Few E2E tests (critical user paths only)

## Test Types

### Unit Tests
**Location:** `tests/unit/`
**Purpose:** Test individual functions and components in isolation
**Run:** `npm test` or `pytest tests/unit`

**What to test:**
- Pure functions (utils, helpers)
- Component rendering
- State management logic
- Data transformations

### Integration Tests
**Location:** `tests/integration/`
**Purpose:** Test API endpoints and component interactions
**Run:** `npm test tests/integration` or `pytest tests/integration`

**What to test:**
- API endpoint responses
- Database operations
- Authentication flows
- Multi-component interactions

### E2E Tests (Optional)
**Location:** `tests/e2e/`
**Purpose:** Test complete user flows
**Run:** `npm run test:e2e`

**What to test:**
- Critical user journeys
- Happy path flows
- Authentication end-to-end

## Test Commands

| Command | Description |
|---------|-------------|
| `npm test` | Run all tests once |
| `npm run test:watch` | Run tests in watch mode |
| `npm run test:coverage` | Run tests with coverage report |
| `npm run test:ui` | Open visual test UI |

## Coverage Goals

**Current:** {current}%
**Target:** 70-80% for critical paths

Focus coverage on:
- Business logic
- API endpoints
- Authentication
- Data validation

## Writing New Tests

### Naming Convention
- Test files: `*.test.ts` or `*.spec.ts`
- Test descriptions: "should [expected behavior] when [condition]"

### Test Structure
```typescript
describe('ComponentName', () => {
  describe('method or behavior', () => {
    it('should do something when condition', () => {
      // Arrange
      // Act
      // Assert
    })
  })
})
```

### Best Practices
1. One assertion per test (when practical)
2. Test behavior, not implementation
3. Use descriptive test names
4. Keep tests independent
5. Mock external dependencies

## CI Integration

Tests run automatically on:
- Every push to `main` and `dev` branches
- Every pull request

See `.github/workflows/ci.yml` for configuration.

{storage_testing_section}

## Resources

- [Testing Library Docs](https://testing-library.com/)
- [Vitest Docs](https://vitest.dev/)
- [Pytest Docs](https://docs.pytest.org/)
```
</strategy-template>

<storage-testing-strategy-section>
Include this section when project uses storage (from deployment-strategy.md):

```markdown
## Storage Testing

**Storage Type:** {Local VPS / Backblaze B2 / Tigris}

### Unit Tests (Mocked)
Location: `tests/unit/storage.test.ts`
Purpose: Test storage logic without external dependencies

- Mock S3 client responses
- Test upload/download logic
- Test error handling
- Test file validation

### Integration Tests (Real Storage)
Location: `tests/integration/storage.test.ts`
Purpose: Verify actual storage connectivity

Prerequisites:
- `TEST_STORAGE_BUCKET` environment variable
- Storage credentials configured

Run: `npm run test:integration` (skipped if env not configured)

### What to Test

**Unit tests (always run):**
- File type validation
- File size limits
- Path sanitization
- Error handling for network failures
- URL generation

**Integration tests (optional, against test bucket):**
- Upload/download roundtrip
- File listing
- Delete operations
- Presigned URL generation

### Environment Variables for Testing

[If Backblaze B2:]
```
TEST_STORAGE_BUCKET=your-test-bucket
B2_APPLICATION_KEY_ID=your-key-id
B2_APPLICATION_KEY=your-key
B2_ENDPOINT=https://s3.region.backblazeb2.com
```

[If Tigris (Fly.io):]
```
TEST_STORAGE_BUCKET=your-test-bucket
# Credentials auto-injected in Fly.io environment
# For local testing, set manually:
AWS_ACCESS_KEY_ID=tid_...
AWS_SECRET_ACCESS_KEY=tsec_...
AWS_ENDPOINT_URL_S3=https://fly.storage.tigris.dev
```

[If Local VPS:]
```
TEST_UPLOAD_DIR=/tmp/test-uploads
```

### Test Bucket Best Practices

1. Use a dedicated test bucket (not production)
2. Configure lifecycle rules to auto-delete old test files
3. Prefix test files with `test-` for easy identification
4. Clean up after tests (afterAll hooks)
```
</storage-testing-strategy-section>
</phase>

<phase id="7" name="provide-guidance">
<action>Summarize what was created and provide educational guidance.</action>

<summary-template>
## Test Infrastructure Setup Complete

**Project:** {project_name}
**Framework:** {test_framework}

---

### Files Created

- {config_file} - Test framework configuration
- tests/setup.{ext} - Test setup and global mocks
- tests/unit/example.test.{ext} - Example unit test
- tests/integration/api.test.{ext} - Example integration test
- .docs/test-strategy.md - Testing strategy documentation

---

### Test Commands

```bash
# Run all tests
{run_command}

# Run tests in watch mode
{watch_command}

# Run with coverage
{coverage_command}
```

---

### What's Next

Your testing infrastructure is set up. Here's how to proceed:

1. **Run the example tests** to verify setup:
   ```bash
   {run_command}
   ```

2. **Write tests as you build features:**
   - Write tests before or alongside new code
   - Focus on critical business logic first
   - Use example tests as templates

3. **Maintain test health:**
   - Keep tests passing
   - Review coverage periodically
   - Update tests when behavior changes

---

### Educational Notes

**Testing Pyramid:**
- Unit tests are fast and numerous - test small pieces in isolation
- Integration tests verify components work together
- E2E tests are slow but valuable for critical paths

**When to write tests:**
- Before implementing (TDD) - helps design better code
- After implementing - validates behavior
- When fixing bugs - prevent regression

**Test-Driven Development (TDD) cycle:**
1. Write a failing test
2. Write minimum code to pass
3. Refactor while keeping tests green

---

### Workflow Status

This is an **optional** phase in the Skills workflow.

**Previous:** Phase 3 (project-spinup) - Project foundation
**Current:** Phase 4 (test-orchestrator) - Test infrastructure
**Next:** Phase 5 (deploy-guide) - Deploy your application

You can continue building features and write tests as you go.
When ready to deploy, use the **deploy-guide** skill.
</summary-template>
</phase>

</workflow>

---

<guardrails>

<must-do>
- Analyze project structure before recommending framework
- Match testing tools to tech stack
- Create working example tests
- Document testing strategy in .docs/test-strategy.md
- Provide educational guidance on testing patterns
- Include commands for running tests
- Create setup files for test environment
- Check .docs/deployment-strategy.md for storage strategy
- Include storage mocking patterns when project uses object storage
- Include storage integration test examples when B2 or Tigris is used
- Document storage testing approach in test-strategy.md when applicable
</must-do>

<must-not-do>
- Write comprehensive test suites (that's ongoing development)
- Force a specific testing approach without consideration
- Skip the strategy documentation
- Create tests that won't run with current project setup
- Recommend E2E testing infrastructure for simple projects
- Skip storage testing scaffolding when storage strategy is defined
- Run integration tests against production storage buckets
</must-not-do>

</guardrails>

---

<outputs>
**.docs/test-strategy.md** — Testing strategy documentation with:
- Testing philosophy and approach
- Test types implemented (unit, integration, e2e)
- Test file organization and commands
- Coverage goals and CI integration notes

**Additional outputs:**
- Test framework configuration files (vitest.config.ts, jest.config.js, pytest.ini, etc.)
- Test scaffolding (example tests, setup files)
- Package scripts for running tests

**Flexible Entry:** This skill analyzes project structure to recommend appropriate testing tools. Upstream handoffs inform but don't block — missing context is gathered through project analysis.
</outputs>
