# NPM Build Template v0.2 - Usage Examples

The `account.npm_build` v0.2 template provides flexible working directory support for various npm project structures. This guide demonstrates how to use the template with different project layouts.

## Template Features

### New in v0.2:
- **Flexible Working Directory**: Support for npm projects in any subdirectory
- **Template Input Configuration**: All commands configurable via template inputs
- **Enhanced Validation**: Workspace and package.json validation before operations
- **Backward Compatibility**: Works with existing v0.1 configurations
- **Resource Optimization**: Configurable resource limits based on project size
- **Security Hardening**: Enhanced audit capabilities with configurable thresholds

## Usage Patterns

### 1. Root-Level NPM Project (Backward Compatible)

For projects where `package.json` is at the repository root:

```yaml
stages:
  - stage:
      name: npm build
      identifier: npm_build
      template:
        templateRef: account.npm_build
        versionLabel: v0.2
        templateInputs:
          type: CI
          spec:
            infrastructure:
              type: KubernetesDirect
              spec:
                connectorRef: org.k8sdocacloud
                namespace: harness-ci
            execution:
              steps:
                - step:
                    identifier: Workspace_Validation
                    spec:
                      connectorRef: org.k8sdocacloud
                      image: node:20
                      # workingDirectory defaults to "."
                - step:
                    identifier: BuildAndPushDockerRegistry_1
                    spec:
                      connectorRef: org.Dockerhub
                      repo: stfgit/my-app
                      dockerfile: Dockerfile
                      context: "."
```

### 2. Frontend Subdirectory Project

For projects where npm project is in a subdirectory like `frontend/`, `client/`, etc.:

```yaml
stages:
  - stage:
      name: npm build frontend
      identifier: npm_build_frontend
      template:
        templateRef: account.npm_build
        versionLabel: v0.2
        templateInputs:
          type: CI
          spec:
            infrastructure:
              type: KubernetesDirect
              spec:
                connectorRef: org.k8sdocacloud
                namespace: harness-ci
            execution:
              steps:
                - step:
                    identifier: Workspace_Validation
                    spec:
                      connectorRef: org.k8sdocacloud
                      image: node:20
                      # Configure working directory
                      workingDirectory: "mower-frontend/"
                - step:
                    identifier: Install_Dependencies
                    spec:
                      installCommand: "npm ci --prefer-offline"
                - step:
                    identifier: Code_Quality_Check
                    spec:
                      lintCommand: "npm run lint:fix"
                - step:
                    identifier: Build_Application
                    spec:
                      buildCommand: "npm run build:prod"
                - step:
                    identifier: Run_Tests
                    spec:
                      testCommand: "npm run test:coverage"
                - step:
                    identifier: BuildAndPushDockerRegistry_1
                    spec:
                      connectorRef: org.Dockerhub
                      repo: stfgit/mower-java-frontend
                      dockerfile: "mower-frontend/Dockerfile"
                      context: "mower-frontend/"
```

### 3. Monorepo Package Structure

For monorepo structures with multiple packages:

```yaml
stages:
  - stage:
      name: build web app
      identifier: build_web_app
      template:
        templateRef: account.npm_build
        versionLabel: v0.2
        templateInputs:
          type: CI
          spec:
            infrastructure:
              type: KubernetesDirect
              spec:
                connectorRef: org.k8sdocacloud
                namespace: harness-ci
            execution:
              steps:
                - step:
                    identifier: Workspace_Validation
                    spec:
                      workingDirectory: "packages/web-app/"
                - step:
                    identifier: Install_Dependencies
                    spec:
                      installCommand: "npm install --workspace=packages/web-app"
                - step:
                    identifier: Build_Application
                    spec:
                      buildCommand: "npm run build --workspace=packages/web-app"
                - step:
                    identifier: BuildAndPushDockerRegistry_1
                    spec:
                      dockerfile: "packages/web-app/Dockerfile"
                      context: "packages/web-app/"
```

### 4. Advanced Configuration with Security Hardening

Enterprise configuration with enhanced security settings:

```yaml
stages:
  - stage:
      name: secure npm build
      identifier: secure_npm_build
      template:
        templateRef: account.npm_build
        versionLabel: v0.2
        templateInputs:
          type: CI
          spec:
            infrastructure:
              type: KubernetesDirect
              spec:
                connectorRef: org.k8sdocacloud
                namespace: harness-security-scan
            execution:
              steps:
                - step:
                    identifier: Workspace_Validation
                    spec:
                      workingDirectory: "frontend/"
                      resources:
                        limits:
                          memory: 1Gi
                          cpu: 500m
                - step:
                    identifier: Install_Dependencies
                    spec:
                      installCommand: "npm ci --audit --no-fund"
                      resources:
                        limits:
                          memory: 3Gi
                          cpu: 1500m
                - step:
                    identifier: Code_Quality_Check
                    spec:
                      lintCommand: "npm run lint -- --max-warnings 0"
                - step:
                    identifier: Run_Tests
                    spec:
                      testCommand: "npm run test:coverage -- --coverage --coverageThreshold=80"
                - step:
                    identifier: Security_Audit
                    spec:
                      auditLevel: "moderate"
                      failOnAudit: "true"
                - step:
                    identifier: BuildAndPushDockerRegistry_1
                    spec:
                      dockerfile: "frontend/Dockerfile.prod"
                      context: "frontend/"
                      optimize: true
```

## Template Input Reference

### Available Input Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `workingDirectory` | String | `"."` | Directory containing package.json |
| `installCommand` | String | `npm install --prefer-offline --no-audit --no-fund` | NPM install command |
| `lintCommand` | String | `npm run lint` | Linting command |
| `buildCommand` | String | `npm run build` | Build command |
| `testCommand` | String | `npm run test:ci` | Test command |
| `auditLevel` | String | `high` | NPM audit severity level (low, moderate, high, critical) |
| `failOnAudit` | Boolean | `false` | Fail pipeline on audit violations |
| `skipLint` | Boolean | `false` | Skip linting step |
| `skipTests` | Boolean | `false` | Skip testing step |
| `skipAudit` | Boolean | `false` | Skip security audit |
| `dockerfile` | String | `Dockerfile` | Path to Dockerfile |
| `dockerContext` | String | `"."` | Docker build context |
| `nodeImage` | String | `node:20` | Node.js Docker image |

### Resource Configuration

Configure resources per step based on project requirements:

```yaml
steps:
  - step:
      identifier: Build_Application
      spec:
        resources:
          limits:
            memory: "8Gi"    # Large builds
            cpu: "4000m"
          requests:
            memory: "4Gi"
            cpu: "2000m"
```

## Integration with Other Templates

### Complete NPM Pipeline with v0.2

Update existing npm_pipeline templates to use v0.2:

```yaml
stages:
  - stage:
      name: npm build and push
      identifier: npm_build_and_push
      template:
        templateRef: account.npm_build
        versionLabel: v0.2  # Updated to v0.2
        templateInputs:
          type: CI
          spec:
            execution:
              steps:
                - step:
                    identifier: Workspace_Validation
                    spec:
                      workingDirectory: <+input>.default(.)
```

## Migration from v0.1 to v0.2

### Backward Compatibility

v0.2 is fully backward compatible with v0.1 configurations. Simply update the `versionLabel`:

```yaml
# Before (v0.1)
template:
  templateRef: account.npm_build
  versionLabel: v0.1

# After (v0.2) - No other changes needed
template:
  templateRef: account.npm_build
  versionLabel: v0.2
```

### Enhanced Configuration

To leverage v0.2 features, add working directory configuration:

```yaml
# Enhanced v0.2 usage
templateInputs:
  type: CI
  spec:
    execution:
      steps:
        - step:
            identifier: Workspace_Validation
            spec:
              workingDirectory: "frontend/"
        - step:
            identifier: BuildAndPushDockerRegistry_1
            spec:
              dockerfile: "frontend/Dockerfile"
              context: "frontend/"
```

## Real-World Examples

### MowItNow Project Structure

For the harness-play project with frontend in subdirectory:

```
harness-play/
├── pom.xml              # Java backend
├── Dockerfile           # Backend Docker
├── mower-frontend/      # NPM project directory
│   ├── package.json
│   ├── Dockerfile       # Frontend Docker
│   └── src/
└── .harness/
```

Configuration:
```yaml
templateInputs:
  type: CI
  spec:
    execution:
      steps:
        - step:
            identifier: Workspace_Validation
            spec:
              workingDirectory: "mower-frontend/"
        - step:
            identifier: BuildAndPushDockerRegistry_1
            spec:
              dockerfile: "mower-frontend/Dockerfile"
              context: "mower-frontend/"
```

### Nextly Project Structure

For root-level npm projects:

```
nextly/
├── package.json         # Root-level NPM project
├── Dockerfile
├── src/
└── .harness/
```

Configuration:
```yaml
templateInputs:
  type: CI
  spec:
    execution:
      steps:
        - step:
            identifier: Workspace_Validation
            spec:
              workingDirectory: "."  # Default, can be omitted
```

## Best Practices

### 1. Working Directory Patterns

- Use relative paths: `"frontend/"` not `"/app/frontend"`
- Include trailing slash for clarity: `"client/"` not `"client"`
- Validate directory exists in your repository

### 2. Docker Configuration

- Match dockerfile and context paths:
  ```yaml
  dockerfile: "frontend/Dockerfile"
  context: "frontend/"
  ```

### 3. Resource Optimization

- Configure resources based on project size:
  - Small projects: 1Gi memory, 500m CPU
  - Medium projects: 2Gi memory, 1000m CPU
  - Large projects: 4Gi+ memory, 2000m+ CPU

### 4. Security Configuration

- Use `failOnAudit: true` for production pipelines
- Set appropriate `auditLevel` based on risk tolerance
- Enable all security steps for enterprise environments

## Troubleshooting

### Common Issues

1. **Working directory not found**
   ```
   ❌ Working directory 'frontend/' does not exist
   ```
   Solution: Verify directory path and ensure it exists in repository

2. **Package.json not found**
   ```
   ❌ package.json not found in /harness/frontend
   ```
   Solution: Ensure package.json exists in specified working directory

3. **Docker context mismatch**
   ```
   Error: Dockerfile not found in context
   ```
   Solution: Align dockerfile and context paths

### Debug Steps

1. Enable workspace validation step logging
2. Verify directory structure in repository
3. Check dockerfile and context path alignment
4. Validate template input parameter values