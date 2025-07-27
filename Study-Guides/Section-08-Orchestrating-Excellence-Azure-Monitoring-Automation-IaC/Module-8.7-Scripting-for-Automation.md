### **💻 Module 8.7: Scripting for Automation - Azure CLI & Azure PowerShell**

While Infrastructure as Code (IaC) tools like Bicep define the desired state of your infrastructure declaratively, there are many scenarios where you need to manage Azure resources imperatively or perform specific operational tasks. 

This is where **Azure CLI** and **Azure PowerShell** come in. These powerful command-line tools provide direct, programmatic access to nearly every Azure service, making them indispensable for scripting, automation, and ad-hoc management.

#### **I. What are Azure CLI and Azure PowerShell?**

Both Azure CLI and Azure PowerShell are command-line tools provided by Microsoft to interact with Azure. 

They allow you to create, manage, and delete Azure resources and perform administrative tasks directly from your terminal or within scripts.

1.  **Azure CLI (Command-Line Interface):**
    * **Nature:** A cross-platform command-line tool designed for managing Azure resources.
    * **Syntax:** Uses a simple, compact, and consistent syntax (e.g., `az <command-group> <command> --parameter <value>`). It's inspired by Linux command-line tools.
    * **Output:** Primarily returns JSON output, which is easy to parse and integrate into other scripting languages or applications.
    * **Supported Shells:** Runs on Bash, PowerShell, and Windows Command Prompt (CMD).

2.  **Azure PowerShell:**
    * **Nature:** A set of PowerShell modules (the `Az` module) built on top of the PowerShell scripting language and framework.
    * **Syntax:** Uses PowerShell's verb-noun cmdlets (e.g., `Get-AzVM`, `New-AzResourceGroup`, `Set-AzStorageAccount`). It leverages PowerShell's object-based pipeline.
    * **Output:** Returns PowerShell objects, which can be easily filtered, sorted, and piped to other cmdlets, making it powerful for complex data manipulation.
    * **Supported Shells:** Runs on PowerShell (Windows PowerShell 5.1 and PowerShell 7.x+ on Windows, Linux, and macOS).

#### **II. Key Differences and When to Use Which**

| Feature/Aspect      | Azure CLI                                | Azure PowerShell                               |
| :------------------ | :--------------------------------------- | :--------------------------------------------- |
| **Primary Audience** | Developers, Linux users, cross-platform users | Windows administrators, PowerShell users, .NET developers |
| **Syntax Style** | Linux-like, compact, simple (`az vm create`)    | Verb-Noun cmdlets (`New-AzVM`)                   |
| **Platform** | Windows, Linux, macOS                    | Windows, Linux, macOS (PowerShell Core)        |
| **Output Format** | Primarily JSON (easy to parse)          | PowerShell objects (easy to pipe and manipulate) |
| **Learning Curve** | Generally quicker to pick up for new users | Steeper for those unfamiliar with PowerShell concepts |
| **Interactive Use** | Excellent                                | Excellent                                      |
| **Scripting** | Great for Bash/Shell scripting            | Excellent for PowerShell scripting             |
| **Consistency** | High consistency across commands          | High consistency with PowerShell ecosystem     |

**When to choose which:**

* **Choose Azure CLI if:**
    * You prefer a concise, Linux-style command-line experience.
    * You are working on non-Windows operating systems (Linux, macOS).
    * You are primarily interested in creating and managing resources and can parse JSON output.
    * Your existing automation heavily uses Bash or other shell scripting.
* **Choose Azure PowerShell if:**
    * You are already familiar and comfortable with PowerShell.
    * You are a Windows administrator primarily working in Windows environments.
    * You need to perform complex data manipulation and piping of objects.
    * You are integrating with other PowerShell-based tools or scripts.

Both tools have near parity in terms of Azure service coverage, and you can often achieve the same outcome with either. Many organizations use both, leveraging each tool's strengths for different scenarios.

#### **III. Installation and Authentication**

**A. Installation:**

* **Azure CLI:** Typically installed via platform-specific installers (MSI for Windows, `apt` for Debian/Ubuntu, `yum` for RHEL/CentOS, `brew` for macOS).
* **Azure PowerShell:** Installed as a PowerShell module (`Az`). You install it using `Install-Module -Name Az -Scope CurrentUser` (or for all users).

**B. Authentication (`az login` / `Connect-AzAccount`):**

The most common way to authenticate for interactive sessions is:

* **Azure CLI:** `az login`
    * Opens a browser for interactive authentication.
    * For automation, use service principals or managed identities.
* **Azure PowerShell:** `Connect-AzAccount`
    * Opens a browser for interactive authentication.
    * For automation, use service principals, managed identities, or managed service identity (MSI).

**Authentication for Automation (Non-interactive):**

* **Service Principals:** Create an Azure AD application registration (service principal) and assign it appropriate roles (e.g., Contributor) to your Azure subscription or resource group.
    * **Azure CLI:** `az login --service-principal -u <appId> -p <passwordOrCert> --tenant <tenantId>`
    * **Azure PowerShell:** `Connect-AzAccount -ServicePrincipal -TenantId <tenantId> -ApplicationId <appId> -CertificateThumbprint <thumbprint>` or `... -Credential $pscredential`
* **Managed Identities:** For Azure resources (like Azure VMs, Azure Functions, Azure Automation accounts), use managed identities to authenticate without managing credentials. This is the most secure and recommended approach for Azure-hosted automation. Both CLI and PowerShell automatically leverage the managed identity of the executing resource.

#### **IV. Common Use Cases for Scripting**

Both tools excel at various administrative and automation tasks:

1.  **Resource Provisioning and Management:**
    * Creating, updating, and deleting resource groups, VMs, storage accounts, VNETs, databases, etc.
    * **Example (Azure CLI):** `az vm create --resource-group myRG --name myVM --image UbuntuLTS --admin-username azureuser --generate-ssh-keys`
    * **Example (Azure PowerShell):** `New-AzResourceGroup -Name "myRG" -Location "EastUS"` followed by `New-AzVM -ResourceGroupName "myRG" -Name "myVM" -Image "UbuntuLTS" -Credential (Get-Credential) -Location "EastUS"`
2.  **Configuration and Updates:**
    * Modifying resource properties (e.g., updating VM size, changing storage account tiers, configuring network security groups).
    * **Example (Azure CLI):** `az vm resize --resource-group myRG --name myVM --size Standard_D2s_v3`
    * **Example (Azure PowerShell):** `Update-AzVM -ResourceGroupName "myRG" -Name "myVM" -VM (Get-AzVM -ResourceGroupName "myRG" -Name "myVM") -Size "Standard_D4s_v3"`
3.  **Querying and Reporting:**
    * Retrieving information about Azure resources, filtering results, and generating custom reports.
    * **Azure CLI:** Powerful `--query` parameter using JMESPath for complex JSON filtering.
    * **Azure PowerShell:** Leverages PowerShell's object pipeline (`Where-Object`, `Select-Object`, `Sort-Object`) for robust data manipulation.
    * **Example (Azure CLI):** `az vm list --query "[?contains(tags.environment, 'prod')].{Name:name, Location:location, Size:hardwareProfile.vmSize}" -o table`
    * **Example (Azure PowerShell):** `Get-AzVM | Where-Object Tags -like "*prod*" | Select-Object Name, Location, @{N='Size'; E={$_.HardwareProfile.VmSize}} | Format-Table`
4.  **Automation of Operational Tasks:**
    * Starting/stopping VMs on a schedule.
    * Automating backups or data transfers.
    * Cleaning up old resources.
    * Responding to alerts (e.g., triggering a script via an Azure Automation runbook).
5.  **Integration with CI/CD Pipelines:**
    * Both tools are heavily used in Azure DevOps Pipelines and GitHub Actions to deploy and manage Azure resources. They are typically invoked within `AzureCLI@2` or `AzurePowerShell@5` tasks.
    * Useful for tasks that are not natively supported by ARM/Bicep (e.g., setting up AAD applications, running pre-deployment checks, post-deployment configurations).

#### **V. Best Practices for Writing Robust and Reusable Scripts**

1.  **Parameterize Your Scripts:**
    * Avoid hardcoding values directly in your scripts. Use parameters for inputs like resource group names, locations, passwords, or configurations specific to an environment.
    * **PowerShell:** Use `param()` block with `[Parameter(Mandatory=$true)]` and `[string]`, `[int]`, etc.
    * **Bash/CLI:** Use shell variables (e.g., `$RG_NAME`, `$LOCATION`) and pass them as arguments.
2.  **Handle Authentication Securely:**
    * Never hardcode credentials. For automation, always use Managed Identities (if running on Azure) or Service Principals with least privilege.
    * For interactive use, `az login` or `Connect-AzAccount` are fine.
3.  **Include Error Handling:**
    * Implement `try-catch` blocks (PowerShell) or `set -e` / `trap` (Bash) to gracefully handle errors and ensure scripts don't fail silently.
    * Log errors and provide informative messages.
    * **PowerShell:** Use `-ErrorAction Stop` on cmdlets when you want failures to throw an exception.
4.  **Idempotence (where applicable):**
    * Design scripts to be idempotent, meaning running them multiple times yields the same result without unintended side effects. For example, check if a resource exists before attempting to create it.
    * This is especially important if your scripts are part of an automated pipeline.
5.  **Use Consistent Naming Conventions:**
    * Apply consistent naming for resources within your scripts.
6.  **Add Comments and Documentation:**
    * Clearly comment your code to explain complex logic or critical sections.
    * Include headers detailing the script's purpose, parameters, and usage examples.
7.  **Output Useful Information:**
    * Use `Write-Host` (PowerShell) or `echo` (Bash) for user-friendly messages.
    * Use `Write-Output` (PowerShell) for structured data that can be consumed by other commands.
8.  **Leverage Pipelining (PowerShell):**
    * PowerShell's object pipeline is incredibly powerful. Instead of processing results with loops, try to pipe objects directly from one cmdlet to the next.
    * **Example:** `Get-AzVM | Stop-AzVM`
9.  **Query Output Effectively:**
    * **Azure CLI:** Master JMESPath with the `--query` parameter to extract specific data from JSON output.
    * **Azure PowerShell:** Utilize `Select-Object` and `Where-Object` to filter and transform the PowerShell objects.
10. **Test Your Scripts Thoroughly:**
    * Test scripts in non-production environments first.
    * Consider Pester (PowerShell) or similar testing frameworks for automated script testing.
11. **Store Scripts in Source Control:**
    * Keep your scripts in a version control system (Git) for collaboration, change tracking, and easy rollback.
12. **Consider Modularity (PowerShell Functions/Modules):**
    * For larger or frequently used logic, encapsulate it into PowerShell functions or modules to promote reusability.

---