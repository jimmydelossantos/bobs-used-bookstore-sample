# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Summary

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run a full NuGet package restore to ensure all dependencies are resolved correctly in the new target framework:

```bash
dotnet restore
```

Review the output for any warnings related to package compatibility or deprecated packages that may need to be updated.

---

## 2. Build the Entire Solution

Perform a clean build of the solution to confirm there are no compilation issues:

```bash
dotnet build --configuration Release
```

Address any warnings that surface during the build, particularly those related to nullable reference types or obsolete APIs, as these can indicate areas of the code that may behave differently under the new runtime.

---

## 3. Run the Unit Tests

Execute the test suite in `Bookstore.Domain.Tests` to verify that domain logic behaves as expected after the migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test results carefully. Any failing tests should be investigated to determine whether they represent regressions introduced by the migration or pre-existing issues.

---

## 4. Validate the Data Layer

Since `Bookstore.Data` handles data access, verify the following:

- **Database provider packages** (e.g., Entity Framework Core) are the correct versions compatible with the new target framework.
- **Migrations** are up to date. Run the following to check the current migration state:

```bash
dotnet ef migrations list --project app/Bookstore.Data
```

- If you are using a connection string, confirm it is correctly configured in the new environment's configuration files (`appsettings.json` or environment variables).

---

## 5. Run the Web Application Locally

Start the `Bookstore.Web` project locally to perform manual validation of the application:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Verify the following:
- The application starts without runtime exceptions.
- Key pages and endpoints load and return expected data.
- Any authentication or session handling works correctly.
- Static assets are served properly.

---

## 6. Validate the CDK Project

The `Bookstore.Cdk` project defines infrastructure. Review the following:

- Confirm that the CDK library packages (e.g., `Amazon.CDK` or similar) are updated to versions compatible with the new target framework.
- Synthesize the CDK stack to verify the infrastructure definition compiles and produces the expected output:

```bash
dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj
```

Or, if using the CDK CLI:

```bash
cdk synth
```

Review the synthesized output for any unexpected changes compared to the previous infrastructure definition.

---

## 7. Review Configuration and Environment Settings

Cross-platform .NET may handle certain configuration sources differently than .NET Framework. Confirm the following:

- `appsettings.json` and `appsettings.{Environment}.json` files are present and correctly structured.
- Any configuration previously stored in `Web.config` or `App.config` has been migrated to the appropriate `appsettings.json` entries or environment variables.
- File path separators and case sensitivity are handled correctly, particularly if the application will run on Linux.

---

## 8. Check for Platform-Specific API Usage

Even without build errors, certain APIs may behave differently or throw at runtime on non-Windows platforms. Search the codebase for usage of:

- `System.Drawing` (not fully supported cross-platform without additional packages)
- Windows Registry access
- Windows-specific file path assumptions
- COM interop

Replace or abstract any such usages with cross-platform alternatives where applicable.