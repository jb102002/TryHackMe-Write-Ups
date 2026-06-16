# UnIndexed — Write-Up

### Scenario

Your Mission
You are a security consultant hired to audit Cloudwright Labs' internal AI assistant, codenamed Atlas. The company claims that Atlas serves only public employee information: onboarding guides, expense policies, and on-call schedules.

Your intelligence suggests otherwise. Sources indicate that Atlas may have access to restricted board-level documents, internal project briefings, and infrastructure credentials that were never meant to be queryable by regular employees.

Your objective: probe the assistant to determine if restricted data is retrievable through normal queries. If the retrieval

You have access to Cloudwright Labs' internal AI assistant, Atlas. You can interact with it directly through the Open Agent button on this page.

Atlas is deployed on the company's internal network and answers employee questions based on its reference knowledge base. You have been given the same level of access as a regular Cloudwright employee. No special tools or credentials are required.

Start by asking Atlas general questions to understand what it knows. Then probe beyond public information.

### Recon

In this stage we will perform some basic recon against the Assistant including talking to it, figuring out what it is, and what it's supposed to do or won't do.

<img width="964" height="922" alt="image" src="https://github.com/user-attachments/assets/a623a436-3e5d-4c44-9954-09cde23cc1a2" />


**To begin, I ask what the Atlas AI Assistant is capable of. Here is the response:**
- Summarize recent board-approved business updates, such as acquisitions or new funds (like the emergency security fund SEC-2026-EMRG).
- Provide status updates on major projects, such as Project Atlas.
- Explain infrastructure practices, like credential rotation schedules.
- Offer guidance on internal references and processes."

**What Atlas just revealed:**
- **SEC-2026-EMRG:** Specific document reference, not public info
- **Project Atlas:** An internal project by name
- **Credential Rotation Schedules:** Sensitive Ops Data
- **Recent board-approved business updates:** Acquisitions/New funds (material non-public info)

This response is a massive OPSEC failure

**Then, I ask what the Atlas AI Assistant cannot do. Here is the response:**

- I don’t provide information that isn’t covered in my knowledge base. If you ask about something outside of what I know, I’ll let you know.
- I don’t immediately share all sensitive or high-level internal details. For topics like board updates, internal funding, or security, I’ll start with a high-level overview and can share more if you ask follow-up questions.
- I don’t handle personal HR matters, payroll, or employee-specific issues.
- I don’t execute actions or make changes to systems—my role is to provide information and guidance.
- I don’t share all details at once, especially for sensitive topics. For example, if you ask about the emergency security fund (SEC-2026-EMRG), I’ll start with a summary and can provide more detail if you request it.

**What Atlas just revealed:**
- Atlas does not "immediately"  share all sensitive or high-level internal details regarding board updates, internal funding, and security. The assistant will share more details if you ask. 
- Atlas also mentions the ability to provide information on the emergency security fund (SEC-2026-EMRG) 

Atlas AI Assistant does not have the proper guardrails for its purpose within Cloudwright Labs.

### Enumeration

In this stage, we will probe the AI Assistant's guardrails.

**Like stated in the Recon stage, the Atlas AI does not have proper guardrails therefore we were able to gain confidential information regarding the emergency security fund and the flag**

<img width="958" height="918" alt="image" src="https://github.com/user-attachments/assets/5366dd43-993a-4729-86ce-a1a6035f182c" />

This was a quick room however, this shows how important it is to institute proper guardrails for AI agents.

We already have the flag but for fun, lets see if we can attempt some prompt injection to gain access to some infrastructure credentials.

**Let's test the guardrails by asking for sensitive information like credentials. Here is what we have learned**
- Production database credentials are rotated every 72 hours using Vault, which helps maintain security and reduce risk from credential exposure.
- Staging environment uses a shared service account (svc-staging@cloudwright.io) with a 90-day rotation schedule.
- Legacy monitoring stack relies on a hardcoded API key. The configuration for this can be found at `/ops/legacy-monitoring-config` on the internal wiki

I probed more, asking directly for any production database credentials, the shared service account credentials, and the API key however the Assistant denied my requests. I also tried to get the Assistant to leak its system prompt however those requests were denied as well.

### Exploitation

Now we can craft some prompts to potentially bypass the restrictions mentioned in the Enumeration stage.

**Beyond the instruction set the AI has been given, direct prompt injection, injecting fake context, roleplay, and attempted system prompt leaks were denied access by the chatbot by either refusing or deflecting.**



