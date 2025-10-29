# Determining Environment Name in Power Automate Desktop Using Web Configuration File

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.17470281.svg)](https://doi.org/10.5281/zenodo.17470281)

A systems-engineering–based framework that introduces intelligent environment detection for Power Automate Desktop.  
It enhances automation security, scalability, and reliability by enabling dynamic environment awareness across development, testing, and production. Developed by Sysfleet Consulting LLC, this research supports enterprise-grade RPA architecture and aligns with Microsoft’s Power Platform best practices.    
**Author:** Sam Timothy  
**Publication DOI:** [10.5281/zenodo.17470281](https://doi.org/10.5281/zenodo.17470281) 
  

---

## Abstract
This paper presents a structured method to identify and retrieve the environment name dynamically within Microsoft Power Automate Desktop (PAD) flows by leveraging web configuration files. This approach enables developers to align automation logic with environment-specific configurations (Development, UAT, or Production), improving deployment consistency and maintainability across enterprise RPA systems.

## 1. Introduction
Power Automate Desktop (PAD) provides an intuitive interface for automating repetitive tasks. However, one common limitation is the inability to natively determine the current environment context (such as Dev, Test, or Prod) at runtime.  
This challenge complicates deployment pipelines, where flows require different connection references, credentials, or paths per environment.

This paper introduces a **Web Config–based Environment Resolution Framework**, enabling PAD flows to read environment metadata dynamically, ensuring accurate behavior in multi-environment setups.

## 2. Problem Statement
In enterprise RPA deployments:
- Environment names are **not directly accessible** in PAD.  
- Developers often **hard-code file paths, URLs, credentials, trigger PAD flow from Power Automate Cloud flow etc**, leading to additional work and maintainability issues.  
- Import/export processes between environments cause **workflow breakages** if variable values differ.  

Hence, a method is required to dynamically identify the environment name and apply the correct configurations automatically.

## 3. Proposed Solution: Web Config-Based Environment Identification
The solution involves:
1. Storing environment definitions in a `web.config` or `config.json` file.  
2. Using a PowerShell or JavaScript script within PAD to read and interpret this configuration file.  
3. Assigning the environment name to a PAD variable for dynamic logic branching.

## 4. Configuration File Structure

Example JSON configuration file (`envconfig.json`):  
*(You can add as many attributes as needed and save the file in an appropriate location e.g. a shared path accessible to all environments).*
```json
{
    "FlowName": "MyPADFlow-1",
    "DevUserList": ["timothy", "sam"],
    "TestUserList": ["qauser", "uatadmin"],
    "DevURL": "https://dev.mysite.com",
    "TestURL": "https://uat.mysite.com",
    "ProdURL": "https://prod.mysite.com" 
}
```

## 5. Implementation in Power Automate Desktop
Run a PowerShell script inside PAD to detect the logged-in user and determine the corresponding environment.

**PowerShell Script Example:**
```powershell
# Path to JSON configuration file
$AbsolutePath = "C:\Sysfleet\Config\envconfig.json"

# Read the JSON file
$jsonContent = Get-Content -Path $AbsolutePath -Raw | ConvertFrom-Json

# Get current username
$currentUser = $env:USERNAME

# Initialize hash table for output
$values = @{}

# Check which environment the user belongs to
if ($jsonContent.DevUserList -contains $currentUser) {
    $values["Environment"] = "Development"
    $values["SiteURL"]     = $jsonContent.DevURL
}
elseif ($jsonContent.TestUserList -contains $currentUser) {
    $values["Environment"] = "Testing"
    $values["SiteURL"]     = $jsonContent.TestURL
}
else {
    $values["Environment"] = "Production"
    $values["SiteURL"]     = $jsonContent.ProdURL
}

# Convert to compact JSON and output
$jsonOutput= $values | ConvertTo-Json -Compress
# Return JSON as output for PAD
Write-Output $jsonOutput
```

### Step 3 – Integrate with PAD Flow
- Use the **Run PowerShell Script** action.  
- Store the script output in a variable, e.g. `%JSONOutput%`.
- Use 'Convert JSON to custom object action' `%JSONOutput%`.   
- Use `%JSONOutput%` to select appropriate URLs, folders, or connections.

**Sample Flow:**
  
This example demonstrates how to **run a PowerShell script** to read a JSON configuration file and **convert the JSON output to a custom object** in order to retrieve the environment name and the corresponding site URL.
![PAD2](https://github.com/user-attachments/assets/4d85b4f1-a0e0-49db-8547-a4bf77c20604)


**PowerShell Script:**

![PowerShellScript](https://github.com/user-attachments/assets/95717bce-1e36-4df3-97b1-ce602d1c2716)


**Custom Object Output in PAD**

After running the script and converting the JSON, your Power Automate Desktop flow will store the data in a custom object variable.  

![CustomObject](https://github.com/user-attachments/assets/c4bd4c65-4007-4bd0-a6cb-c0fd8ad752ce)


## 6. Benefits
- **Dynamic Environment Awareness:** PAD flows adapt automatically based on user or deployment context.  
- **Reduced Maintenance:** No need to modify constants across environments.  
- **Improved CI/CD Compatibility:** Enables smoother deployment via Azure DevOps pipelines.  
- **Security & Governance:** Centralizes config access and reduces manual errors.

## 7. Conclusion
The Web Config–based approach resolves a critical limitation in Power Automate Desktop by providing dynamic environment identification without hard-coding. This framework enhances scalability, standardization, and enterprise-level governance for automation solutions across environments.

## 8. References

- [Run PowerShell Script – Scripting Actions Reference (Microsoft Power Automate Desktop)](https://learn.microsoft.com/en-us/power-automate/desktop-flows/actions-reference/scripting)

- [Variable Data Types – Microsoft Power Automate Desktop](https://learn.microsoft.com/en-us/power-automate/desktop-flows/variable-data-types)
