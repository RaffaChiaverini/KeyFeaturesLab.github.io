---
date: 2025-06-23 02:30:00
layout: post
title: Automating Threat Intelligence with Azure OpenAI & Security Copilot 
subtitle: 'By Raffaele Chiaverini'
description: >-
    A hands-on walkthrough of how to automate threat intelligence extraction and analysis using Azure OpenAI, Microsoft Security Copilot, and Logic Apps—designed to save time, reduce costs, and operationalize security insights.
image: /assets/images/SecCopLogicApp/202AI-PoweredCybersecurityWorkflow_simple_compose.png
optimized_image: /assets/images/SecCopLogicApp/202AI-PoweredCybersecurityWorkflow_simple_compose.png
category: blog
tags:
  - cybersecurity
  - Azure Logic App
  - Azure OpenAI
  - Microsoft Security Copilot
  - blog
author: Raffaele Chiaverini
paginate: true
---


## Introduction

As I explored ways to streamline threat intelligence processing in my own work, I developed a project that helped me deeply understand the real-world potential of two powerful Microsoft technologies: **Security Copilot** and **Azure Logic Apps**. Through this project, I experienced firsthand how these tools can work together to automate complex workflows, reduce manual workload, and deliver faster, more reliable security outcomes.

**Microsoft Security Copilot** is an AI-powered assistant that combines the scale of large language models with Microsoft’s security expertise. It helps analysts interpret threat data, contextualize IOCs, and generate incident summaries with speed and precision.

On the other hand, **Azure Logic Apps** is Microsoft’s low-code automation platform. It allows users to build workflows that react to events, process data, and trigger actions across different systems. When integrated with Security Copilot, Logic Apps can operationalize AI-driven insights automatically parsing threat reports, submitting prompts, and distributing results without human intervention.

In this blog post, I want to share the project that brought these two technologies together—a solution that helped me go from theory to practical understanding, and that has the potential to save organizations significant time and effort in their security operations.

<img src="/assets/images/SecCopLogicApp/secuirty copilot logo.png" 
     alt="placeholder" 
     title="Security Copilot logo" 
     style="width: 300px;">

## My Goal: Automate and Optimize Threat Intelligence Processing

This Azure Logic App is designed to reduce the time and manual effort required to process threat intelligence reports. It enables organizations to:
1. **Automatically extract IOCs from raw reports using Azure OpenAI:** Parses unstructured PDF documents to isolate relevant Indicators of Compromise, eliminating the need for manual triage.
2. **Leverage Security Copilot for scalable, contextual threat evaluation:** Processes extracted IOCs in segments to produce meaningful threat assessments, enabling faster decision-making with minimal analyst input.
3. **Generate structured reports and automate delivery:** Consolidates outputs into readable files and distributes them via email and cloud storage, reducing reporting time from hours to minutes.

# Prerequisites

To implement this solution, the following components are required:
1. **An active Azure subscription:** Necessary to provision and manage the Logic App and related services.
2. **An Azure OpenAI model deployment:** Required for natural language processing tasks, including extracting Indicators of Compromise (IOCs) from unstructured text.
3. **A Microsoft Security Copilot capacity:** Needed to contextualize extracted IOCs and generate threat intelligence insights and response recommendations.

## Why Use Azure OpenAI in the First Place?

Before diving into the Azure Logic App workflow, it’s important to address a key question: Why incorporate an Azure OpenAI deployment when Security Copilot already offers advanced security analysis capabilities?
The answer lies in understanding the purpose and strengths of each tool. **Microsoft Security Copilot** is designed to serve as a high-value assistant for security teams, an intelligent system capable of correlating complex signals, identifying patterns, and delivering actionable insights across the full breadth of an organization’s security environment. Its strength is not in handling basic, repetitive parsing tasks, but in providing strategic, contextualized analysis that supports faster, smarter decisions.

On the other hand, extracting raw Indicators of Compromise (IOCs), vulnerabilities, or structured threat data from unstructured, verbose reports is a low-level but necessary task, one that does not leverage the full analytical potential of Security Copilot and may lead to unnecessary compute costs when handled directly by it.

This is where **Azure OpenAI** comes into play. By using a lightweight, general-purpose model such as o4-mini, we can efficiently extract structured threat intelligence data in a cost-effective and scalable way. Once this extraction is complete, **Security Copilot is brought in to do what it does best:** interpret, contextualize, and enrich the extracted data, uncover emerging attack patterns, evaluate relevance based on real-world threat scenarios, and even propose proactive security campaigns that organizations can adopt globally.

In short, this dual-layered approach allows us to:
* **Offload repetitive, extractive tasks to a lightweight model,** optimizing cost and speed.
* **Reserve Security Copilot for high-impact analysis,** where its capabilities provide the most strategic value.
This isn’t about replacing or working around Security Copilot, it’s about enabling it to be even more effective by using it where it matters most.

## The solution:

<img src="/assets/images/SecCopLogicApp/LAstructure.png" 
     alt="placeholder" 
     title="Logic App structure" 
     style="width: 400px;">

### Azure OpenAI API Key Handling: Best Practices

It is important to begin by addressing a security consideration regarding the handling of the Azure OpenAI API key. Storing the API key in plain text, as shown in the example, is not a recommended practice. Exposing API keys in clear text increases the risk of unauthorized access, especially if the code is stored in version control systems, shared across teams, or accidentally leaked. Such exposure could allow attackers to misuse the key, leading to data breaches, service abuse, or unexpected charges.
For secure key management, consider the following options based on your Logic App configuration:
* **Logic App:** Use the Azure Key Vault connector to securely retrieve and manage the API key.

<img src="/assets/images/SecCopLogicApp/AzureKeyVaultConnector.png" 
     alt="placeholder" 
     title="Azure Key Vault connector" 
     style="width: 800px;">

* **Consumption Logic App:** Utilize Azure Blob Storage to store and access the key securely.

<img src="/assets/images/SecCopLogicApp/Blobconnector.png" 
     alt="placeholder" 
     title="Azure Blob Storage connector" 
     style="width: 800px;">

* **Manual Input:** If neither option is feasible, inputting the key manually at each execution is a more secure alternative to hardcoding it.
Adopting secure key management practices is essential to maintaining the integrity and confidentiality of your automation workflows.

### Extracting Text from PDF Files for Threat Intelligence Analysis

<img src="/assets/images/SecCopLogicApp/textExtractionFromOD.png" 
     alt="placeholder" 
     title="Text extraction from OneDrive" 
     style="width: 400px;">

Following variable initialization, the first operational step focuses on extracting text from Threat Intelligence (TI) reports to enable further analysis.
To analyze these documents effectively, we must first extract their textual content so it can be processed by the Azure OpenAI model to identify Indicators of Compromise (IOCs). In this implementation, all TI reports are stored in PDF format within a OneDrive folder. However, the solution is flexible and can be adapted to work with documents accessed via HTTP links or stored locally.

Using a "For Each" loop, the Logic App iterates over all files in the OneDrive folder. For each file, the workflow:
1. Validates the file type, ensuring it is in a readable format (e.g., PDF, JPEG, CSV, PNG, TIFF, or Word documents).
2. Extracts text content using the "Parse a document" action.
3. Creates a new .txt file containing the extracted text for downstream processing.
This step ensures that raw, unstructured data is converted into machine-readable text, which can then be passed to Azure OpenAI for IOC extraction.

### IOC extraction 

Once the .txt files are created and stored in OneDrive, the next phase involves processing their content to extract Indicators of Compromise (IOCs). To begin, each text file is retrieved from OneDrive one at a time, and the content is passed through the "Chunk text" action. This step ensures that the document is split into smaller, uniform segments to avoid exceeding the maximum token limit supported by the OpenAI model for a single prompt.

<img src="/assets/images/SecCopLogicApp/forEachFolder.png" 
     alt="placeholder" 
     title="Iterate on all the files in the OneDrive folder" 
     style="width: 1000px;">

For this implementation, I used the o4-mini model deployed via Azure OpenAI, as it offered an ideal balance between token capacity and computational efficiency. It was capable of reliably extracting relevant IOCs and generating concise but informative descriptions. The maximum token limit for the model was confirmed via the Azure AI Studio (formerly Azure Foundry) portal, which provides these technical details in a clear format.

<img src="/assets/images/SecCopLogicApp/OpenAITokenCount.png" 
     alt="placeholder" 
     title="Where to find the maximum numvber of tokens supported by the OpenAI model" 
     style="width: 1000px;">

After the text has been successfully segmented, each chunk is processed by the OpenAI model through an HTTP POST request. To perform this step, I followed a guide by Antonio Formato (referenced below), which outlines how to interact with Azure OpenAI deployments over HTTP. 

<img src="/assets/images/SecCopLogicApp/HTTPCall.png" 
     alt="placeholder" 
     title="Where to find the maximum numvber of tokens supported by the OpenAI model" 
     style="width: 1000px;">

The base URI used for this call follows a consistent structure, with three customizable components based on your specific configuration:
* **RESOURCE_NAME:** The name of your Azure OpenAI resource.
* **DEPLOYMENT_ID:** The name of your model deployment (in this case, o4-mini).
* **API_VERSION:** The version of the OpenAI API you are targeting, typically available in the "Code View" section of the deployment portal.

The full request URL takes the following form:
<pre>
  <span style="color:rgb(30, 138, 84);">// Sample link structure for Azure OpenAI HTTP requests</span>
  https://<span style="color:rgb(116, 191, 241);">RESOURCE_NAME</span>.openai.azure.com/openai/deployments/<span style="color: rgb(116, 191, 241);">DEPLOYMENT_ID</span>/chat/completions?api-version=<span style="color: rgb(116, 191, 241);">API_VERSION</span>      
</pre>

To craft an effective prompt, I used the **Prompt Coach agent** available in Microsoft 365 Copilot. This tool proved highly useful in generating clear and purposeful instructions for the model, improving the relevance and clarity of the output. I strongly recommend exploring Prompt Coach if you have not already, as it can significantly streamline prompt development for OpenAI models. The exact prompt I used is provided in the screenshot below.

<pre>
  <span style="color:rgb(30, 138, 84);">// The Security Copilot prompt</span>
  Extract indicators from the following threat intelligence report to create cybersecurity campaigns for study and analysis. 
  Extract the following kind of information from the text: 
    1. CVEs mentioned and the impacted technologies
    2. Indicators of compromise (IOCs)
    3. Tools or malware referenced
    4. Threat type or classification
    5. Targeted countries or sectors
  
  Be clear about all of the extracted information and label it.  I need to write a report, so I need yu to give context to everything you extract. 
  ...[Text extracted from the TI report files]...       
</pre>

Once the model returns its response, the Logic App needs to isolate the message body, excluding all other metadata or structural parameters. To do this, I passed the response through a "Parse JSON" action using a schema generated with the help of Microsoft 365 Copilot. This allowed me to extract only the relevant content from the model’s output and use it for further processing.

<img src="/assets/images/SecCopLogicApp/ParseJSON.png" 
     alt="placeholder" 
     title="Parse JSON response from the OpenAI model" 
     style="width: 1000px;">

### Prompting Security Copilot 

After successfully extracting all relevant Threat Intelligence data from the source reports, the next step is to evaluate and contextualize this data using Microsoft Security Copilot. To maintain consistency and efficiency, I followed the same approach used when interacting with the Azure OpenAI model: the extracted text was divided into manageable segments, each limited to 15,000 tokens, the maximum allowed for a single Security Copilot prompt.

<img src="/assets/images/SecCopLogicApp/ThepromptSecCop.png" 
     alt="placeholder" 
     title="Prompting Security Copilot" 
     style="width: 800px;">

Each chunk of text was then submitted to Security Copilot using the built-in **"Submit a Security Copilot prompt"** action within Logic Apps. The prompt used for this interaction was once again crafted with the assistance of the Prompt Coach agent, ensuring that the instructions provided to the model were clear, targeted, and optimized for generating meaningful analysis.

<pre>
  <span style="color:rgb(30, 138, 84);">// The Security Copilot prompt</span>
  You are a cybersecurity expert. You will receive a cybersecurity report from external sources, analyze the content and extract the following information:
    1. CVEs mentioned and the impacted technologies
    2. Indicators of compromise (IOCs)
    3. Tools or malware referenced
    4. Threat type or classification
    5. Targeted countries or sectors

  You must be complete and exhaustive. Summarize the data and add security value to it.
  At the end of the analysis, generate a summary to support the creation of new threat intelligence campaigns.
  Include a discussion of the potential campaign based on the extracted data. This section should be clearly separated from the CSV output. 
  Remember to make the response clear and easily readible.
  Here you have the report:
  ...[Text extracted from the previous step]...       
</pre>

For each response returned by Security Copilot, the output text was appended to a variable named SecurityCopilotResponse. This method allowed me to accumulate all analytical outputs into a single, unified file, containing comprehensive evaluations of the extracted IOCs. The final result is a consolidated report that reflects the full scope of Security Copilot's analysis across the entire dataset.

<img src="/assets/images/SecCopLogicApp/promptSecurityCopilot.png" 
     alt="placeholder" 
     title="Prompting Security Copilot" 
     style="width: 300px;">

### The final steps 

As the final output of the entire evaluation process, a new file is generated for each analyzed report and stored in OneDrive. Initially, I chose to save these files in plain text (.txt) format for clarity and ease of review. However, this part of the Logic App is highly customizable. Once the final analysis is obtained, the report can be formatted in various ways, including PDF, CSV, or plain text, and stored across multiple destinations such as OneDrive, Google Drive, or even a local directory, depending on operational requirements.

To complete the workflow, the Logic App also sends an email notification to the designated SOC operator (in this case, myself in the demo environment). The email includes both the full report and a direct link to the OneDrive folder containing the newly generated documents, enabling immediate access and further action if needed.

<img src="/assets/images/SecCopLogicApp/thefinalsteps.png" 
     alt="placeholder" 
     title="Prompting Security Copilot" 
     style="width: 400px;">

## An Additional Layer of Optimization: Generating a STIX File

To meet a specific client requirement, I extended the Logic App workflow to support the automatic generation of **STIX-formatted threat intelligence.** This enhancement enables the structured extraction of IOCs directly from the source reports into a format widely used for cyber threat intelligence sharing and automation.

By outputting the extracted data in **STIX (Structured Threat Information Expression)** format, organizations can immediately integrate the results into existing detection systems, create custom detection rules, and proactively strengthen their defenses based on the intelligence surfaced in the reports.
Later in this article, I’ll walk through a sample execution of the Logic App, including a concrete example of a STIX file generated as part of the workflow.

## Cost analysis 

One of the key advantages of this solution is its minimal operational cost, despite the significant value it offers to security teams. Below is a breakdown of the primary cost components involved in running the workflow:
1. **Azure OpenAI:** The cost per call to an Azure OpenAI model is extremely low. During development, I conducted multiple test and debug executions of the Logic App, generating several model prompts, and the total usage cost remained under €0.30 up to mid-June, when the Logic App was finalized.

<img src="/assets/images/SecCopLogicApp/AzureOpenAIcosts.png" 
     alt="placeholder" 
     title="Azure OpenAI costs analysis" 
     style="width: 1000px;">

2. **Security Copilot:** Pricing for Security Copilot is based on the number of Security Compute Units (SCUs) allocated per user per hour. For this implementation, I provisioned 2 SCUs per hour, with 2 additional units reserved for potential overhead. Even when processing lengthy reports spanning multiple pages, the Logic App consistently remained below 1.8 SCUs per execution.

<img src="/assets/images/SecCopLogicApp/SecCopCosts.png" 
     alt="placeholder" 
     title="Azure OpenAI costs analysis" 
     style="width: 1000px;">

3. **Azure Logic App:** The execution cost of a single Logic App workflow is €0.000111. Given the lightweight nature of the actions involved (text extraction, chunking, HTTP calls, and storage), the cumulative impact on cost remains negligible.

In summary, the workflow demonstrates that it's possible to **achieve meaningful security automation and insight at a very low cost,** making it an accessible and scalable option for organizations of all sizes.

## Cost optimization 

If you're interested in minimizing the cost of Security Copilot prompts, I recommend reading a detailed article by my colleague Stefano Pescosolido, linked in the references section below. His work provides a comprehensive look into best practices for managing SCU (Security Compute Unit) consumption.
One of the most effective cost-saving techniques involves **the explicit invocation of Security Copilot's Direct Skill** to be used within the prompt. By doing so, the model can bypass its internal capability selection process, thereby reducing the computational overhead required to understand and route the request. This direct approach results in a more efficient use of resources.

However, in the context of the Logic App workflow described in this article, I intentionally opted to submit an **open-ended prompt** to Security Copilot. While this may result in slightly higher SCU usage, it consistently produced **more comprehensive and accurate analyses of the extracted content.** Importantly, even with this approach, SCU consumption remained well within reasonable and sustainable limits.
Ultimately, this trade-off between precision and performance versus strict cost minimization should be evaluated based on your specific operational needs and the complexity of your use case.

## Execution example 

To demonstrate the Logic App workflow in a real-world scenario, I tested it against the **UNIT 42** report titled *“Blitz Malware: A Tale of Game Cheats and Code Repositories”*, published by Palo Alto Networks. Released within the past year, this Threat Intelligence report provides an in-depth analysis of Blitz, a newly identified Windows-based malware family. 
<pre>
  <span style="color:rgb(30, 138, 84);">// You can find the original report at the following link:</span>
  <a href="https://unit42.paloaltonetworks.com/blitz-malware-2025/" 
     target="_blank" 
     style="color: rgb(116, 191, 241); text-decoration: none;"> 
  https://unit42.paloaltonetworks.com/blitz-malware-2025/</a>       
</pre>


The original document spans 41 pages, containing a significant amount of unstructured data. The Logic App successfully processed the report using:
* **4 calls to Azure OpenAI** for extracting Indicators of Compromise (IOCs)
* **1 call to Microsoft Security Copilot** to analyze the extracted data and generate a contextual summary

The outcome included a comprehensive Security Copilot analysis and a structured **STIX-formatted threat intelligence file.**
Below, I have included::
1. The full Security Copilot analysis output
2. The generated STIX file

<pre>
  You may click on the files' text to access the documents in OneDrive

  <span style="color:rgb(30, 138, 84);">// The full Security Copilot analysis output</span>
  <a href="https://1drv.ms/t/c/138b81fdc97c50a8/EQt6mMNK_21OmNA3DhFH1zcBS1q7hiZokDtCktjfSTDv2w?e=Vc0Pg4" 
     target="_blank" 
     style="color: rgb(116, 191, 241); text-decoration: none;">
  ### Detailed Cybersecurity Report Analysis
  </a>  

  <span style="color:rgb(30, 138, 84);">// The generated STIX file </span>
  <a href="https://1drv.ms/u/c/138b81fdc97c50a8/ERy0-fLH_zdJlaV19ekTxTYBYvYkkeqA1xr4ErQPF3EjQQ?e=2Txuy3" 
     target="_blank" 
     style="color: rgb(116, 191, 241); text-decoration: none;">
  {"type":"bundle","id":"bundle--d4f7e6a4-f3c1-4c78-8da0-123456789abc","objects":[{"type":"threat-actor","id":"threat-actor--1a2b3c4d-5e6f-7890-ab12-cdef34567890","name":"sw1zzx","aliases":["sw1zzx_dev"],"description":"Malware operator and developer of Blitz malware, distributes via Telegram channel @sw1zzx_dev","created":"2025-05-01T00:00:00Z","modified":"2025-05-01T00:00:00Z"},{"type":"malware","id":"malware--2b3c4d5e-6f78-9012-ab34-cdef56789012","name":"Blitz downloader","is_family":false,"description":"First-stage downloader component of Blitz malware, installed as ieapfltr.dll","labels":["downloader"],"created":"2025-04-12T08:40:00Z","modified":"2025-04-12T08:40:00Z"},{"type":"malware","id":"malware--3c4d5e6f-7890-1234-bc56-def678901234","name":"Blitz bot","is_family":false,"description":"Second-stage bot payload of Blitz malware implementing keylogging, screenshot capture, DoS, file transfer","labels":["remote-access-tools"],"created":"2025-04-09T09:52:00Z","modified":"2025-04-09T09:52:00Z"},{"type":"malware","id":"malware--4d5e6f78-9012-3456-cd78-ef8901234567","name":"XMRig miner","is_family":true,"description":"Monero cryptocurrency miner payload downloaded by Blitz bot","labels":["coin-miner"],"created":"2025-04-09T00:00:00Z","modified":"2025-04-09T00:00:00Z"},{"type":"malware","id":"malware--5e6f7890-1234-5678-de90-1234567890ab","name":"cleaner.exe","is_family":false,"description":"Blitz removal tool provided by operator to remove Blitz downloader and persistence","labels":["remediation"],"created":"2025-05-02T00:00:00Z","modified":"2025-05-02T00:00:00Z"},{"type":"attack-pattern","id":"attack-pattern--6f789012-3456-7890-ef12-345678901234","name":"Registry Run Keys / Startup Folder: Logon Scripts","external_references":[{"source_name":"mitre-attack","external_id":"T1037.001","url":"https://attack.mitre.org/techniques/T1037/001"}],"created":"2025-04-25T00:00:00Z","modified":"2025-04-25T00:00:00Z"},{"type":"indicator","id":"indicator--7f891234-5678-90ab-cd12-34567890abcd","name":"Blitz C2 Hugging Face Space Domain","pattern_type":"stix","pattern":"[domain-name:value = 'e445a00fffe335d6dac0ac0fe0a5accc-9591beae439b860-b5c7747.hf.space']","valid_from":"2025-04-25T00:00:00Z","labels":["c2","malicious-activity"]},{"type":"indicator","id":"indicator--8a901234-5678-90ab-cd23-4567890abcde","name":"Previous Blitz C2 Hugging Face Space","pattern_type":"stix","pattern":"[domain-name:value = 'swizxx-blitz-net.hf.space']","valid_from":"2024-12-01T00:00:00Z","labels":["c2","malicious-activity"]},{"type":"indicator","id":"indicator--9b012345-6789-0abc-de34-567890abcdef","name":"External C2 IP Address","pattern_type":"stix","pattern":"[ipv4-addr:value = '176.65.137.44']","valid_from":"2025-04-25T00:00:00Z","labels":["c2","malicious-activity"]},{"type":"indicator","id":"indicator--0c123456-7890-abcd-ef45-67890abcdef1","name":"Pastebin FSziK5eW","pattern_type":"stix","pattern":"[url:value = 'https://pastebin.com/raw/FSziK5eW']","valid_from":"2025-04-25T00:00:00Z","labels":["download","malicious-activity"]},{"type":"indicator","id":"indicator--1d234567-890a-bcde-f123-4567890abcdef","name":"Pastebin RzLEd17Z","pattern_type":"stix","pattern":"[url:value = 'https://pastebin.com/raw/RzLEd17Z']","valid_from":"2025-04-25T00:00:00Z","labels":["download","malicious-activity"]},{"type":"indicator","id":"indicator--2e345678-9012-cdef-2345-67890abcdef2","name":"Paste.rs ABNe6","pattern_type":"stix","pattern":"[url:value = 'https://paste.rs/ABNe6']","valid_from":"2025-04-25T00:00:00Z","labels":["download","malicious-activity"]},{"type":"indicator","id":"indicator--3f456789-0123-def4-3456-7890abcdef34","name":"Catbox tmcbms.dll","pattern_type":"stix","pattern":"[url:value = 'https://files.catbox.moe/tmcbms.dll']","valid_from":"2025-04-25T00:00:00Z","labels":["download","malicious-activity"]},{"type":"indicator","id":"indicator--4a567890-1234-ef56-4567-890abcdef345","name":"Catbox 5byj86","pattern_type":"stix","pattern":"[url:value = 'https://files.catbox.moe/5byj86']","valid_from":"2025-04-25T00:00:00Z","labels":["download","malicious-activity"]},{"type":"indicator","id":"indicator--5b678901-2345-f678-5678-90abcdef4567","name":"Telegram Channel sw1zzx_dev","pattern_type":"stix","pattern":"[url:value = 'https://t.me/sw1zzx_dev']","valid_from":"2025-02-27T00:00:00Z","labels":["distribution","malicious-activity"]},{"type":"observed-data","id":"observed-data--2c345678-90ab-cdef-1234-567890abcdef","first_observed":"2025-04-25T00:00:00Z","last_observed":"2025-04-25T00:00:00Z","number_observed":1,"objects":{"0":{"type":"file","name":"Nerest_CrackBy@sw1zzx_dev.zip","hashes":{"SHA-256":"14467edd617486a1a42c6dab287ec4ae21409a5dc8eb46d77b853427b67d16d6"}},"1":{"type":"file","name":"Elysium_CrackBy@sw1zzx_dev.zip","hashes":{"SHA-256":"056fb07672dac83ef61c0b8b5bdc5e9f1776fc1d9c18ef6c3806e8fb545af78c"}},"2":{"type":"file","name":"ieapfltr.dll","hashes":{"SHA-256":"0e80fe5636336b70b1775e94aaa219e6aa27fcf700f90f8a5dd73a22c898d646"}},"3":{"type":"file","name":"Blitz bot payload","hashes":{"SHA-256":"ae2f4c49f73f6d88b193a46cd22551bb31183ae6ee79d84be010d6acf9f2ee57"}},"4":{"type":"file","name":"XMRig miner","hashes":{"SHA-256":"47ce55095e1f1f97307782dc4903934f66beec3476a45d85e33e48d63e1f2e15"}},"5":{"type":"file","name":"Previous Blitz payload","hashes":{"SHA-256":"abcc59ab11b6828ad76a4064d928b9d627a574848a5a6e060b22cb27cd11b015"}}}}]}
  </a>         
</pre>


## Conclusion

This project demonstrates how combining Azure OpenAI and Microsoft Security Copilot within an Azure Logic App can significantly streamline the processing of threat intelligence reports, **transforming hours of manual effort into a fully automated, scalable workflow.** By delegating repetitive extraction tasks to a lightweight OpenAI model and using Security Copilot for deeper contextual analysis, the solution remains both cost-efficient and operationally impactful. It serves as a practical example of how modern AI tools can be orchestrated to enhance SOC efficiency and reduce response times.

If you’re interested in deploying this automation or would like to learn more about the approach, feel free to reach out to me through the channels available on **KeyFeaturesLab.**

<img src="/assets/images/KeyFeaturesLabAvatar.png" 
     alt="placeholder" 
     title="Azure OpenAI costs analysis" 
     style="width: 200px;">

## References
<ul>
  <li><a href="https://medium.com/@antonio.formato/consume-azure-openai-api-within-microsoft-azure-logic-apps-ed1258219684" style="color: rgb(74, 179, 221);">Antonio Formato – Consume Azure OpenAI API within Microsoft Azure Logic Apps</a></li>
  <li><a href="https://www.linkedin.com/pulse/impact-direct-skill-invocation-automations-stefano-pescosolido-o3yef/?trackingId=0WKGLbETqJjlIgMdYoHnBg%3D%3D" style="color: rgb(74, 179, 221);">Stefano Pescosolido – The Impact of Direct Skill Invocation in Automation</a></li>
</ul>

