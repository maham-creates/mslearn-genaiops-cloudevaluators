# Cloud evaluation workflow

## Prerequisites

### Azure Authentication

This project requires Azure authentication to run the evaluation workflow. The GitHub Actions workflow uses Azure OpenID Connect (OIDC) to authenticate with your Azure AI project.

**If you don't have Azure access:**
- Create a [personal Azure account](https://azure.microsoft.com/free/) (free tier available)
- Create an Azure Service Principal and add its credentials as GitHub secrets:
  - `AZURE_CREDENTIALS` (JSON format containing: clientId, clientSecret, subscriptionId, tenantId)
- Update the workflow to use credential-based auth instead of OIDC (see `azure/login` action documentation)

**For university accounts:**
- Contact your IT department to request Azure access for student projects
- Ask them to set up the federated identity credential for GitHub Actions OIDC
- Ensure your app registration has the correct subject claim: `repo:maham-creates/mslearn-genaiops-cloudevaluators:ref:refs/heads/main`

### Required Environment Variables

- `AZURE_AI_PROJECT_ENDPOINT` - Your Azure AI Foundry project endpoint
- `AZURE_CLIENT_ID` - Azure service principal client ID
- `AZURE_TENANT_ID` - Azure tenant ID
- `AZURE_SUBSCRIPTION_ID` - Azure subscription ID

## Workflow Steps

1. Prepare Dataset
2. Define Evaluation Criteria (Evaluators)

    Upload the JSONL evaluation dataset to Azure AI Foundary and return its ID. The dataset is a list of JSON objects each containing
     - query: user question sent to agent
     - response: agent's answer
     - ground truth: expected correct answer (used by some evaluators)

3. Create Evaluator Defination

    Register an evaluation defination in Foundary. This tells the platform:
     - What data scheme to expect (query/response/grouth truth)
     - Which built in evaluators to run and how to map dataset fields to them

4. Run Evaluation against Dataset
    Start a cloud evaluation run that scores every record in the dataset.

5. Poll for Completion
    Repeatedly check the run status every 10 seconds until it is 'completed'.
    Returns the final run object (which contains the report URL and results).
    Exits with code 1 if the run fails so CI pipelines surface the error.

    Polling is required because cloud evaluations run asynchronously.  
    Your script must keep checking the run status until Foundry finishes all parallel evaluation work.

6. Retrieve & Interpret Results
7. Analyze and Document Findings

     Fetch per-item evaluator outputs, compute aggregate statistics, print a
     human-readable summary, and write the same summary to RESULTS_FILE.

     Scores are on a 1-5 scale; a score >= 3 is considered a pass.

     The written file is intended to be committed to the branch so the
     GitHub Actions workflow can read it without re-running the evaluation.

     Returns the raw list of output items for any further inspection.

Next steps:

   1. Review detailed results in Azure AI Foundry portal
   2. Analyze patterns in successful and failed evaluations
   3. Commit evaluation_results.txt and push so the PR workflow can use it
