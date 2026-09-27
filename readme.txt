GENAI155
Lab Information
Lab Manual Last Updated Date	September 25, 2026
Last Tested Date and Time	September 25 ,2026, 04:28:42
Completion in Last 24 Hours	1333
Overview
In this challenge lab, you will act as an engineer on a project to develop an AI agent for Cymbal Pools, a swimming pool installation company. Your goal is to evaluate, debug, and improve an AI agent designed to manage pool installation records in Google Cloud.
To achieve this, you will:
• Set up a development environment using Agent Development Kit (ADK).
• Create custom evaluation sets with rubrics to assess specific issues of an agent's behavior.
• Improve agent behavior by simplifying tools and updating agent instructions.
• Re-run evaluations to verify improvement.
Challenge Scenario

Cymbal Pools is a small business that installs and maintains pools across the United States.
Your company has been brought on to help upgrade their internal systems to make use of new agentic technology.
Your Challenge
You have been handed a prototype ADK agent that accesses Cymbal Pools' customer database. However, initial tests show that the agent has serious logical flaws:
• Data Inconsistency: It sometimes deletes customer records from one table without adding them to another, causing records to be lost.
• Invalid Transitions: It moves customers between stages in an invalid order (e.g., marking a pool installation as "paid and closed" before construction is even completed!).
Your challenge is to:
	1. Build an automated evaluation set to expose these flaws.
	2. Write robust custom Python tools to enforce consistent transactions and valid transitions.
	3. Restrict the agent's tool access to only use these verified tools.
	4. Re-run evaluations to prove your fixes work.
Setup and requirements
Before you click the Start Lab button
Read these instructions. Labs are timed and you cannot pause them. The timer, which starts when you click Start Lab, shows how long Google Cloud resources are made available to you.
This hands-on lab lets you do the lab activities in a real cloud environment, not in a simulation or demo environment. It does so by giving you new, temporary credentials you use to sign in and access Google Cloud for the duration of the lab.
To complete this lab, you need:
• Access to a standard internet browser (Chrome browser recommended).
Note: Use an Incognito (recommended) or private browser window to run this lab. This prevents conflicts between your personal account and the student account, which may cause extra charges incurred to your personal account.
• Time to complete the lab—remember, once you start, you cannot pause a lab.
Note: Use only the student account for this lab. If you use a different Google Cloud account, you may incur charges to that account.
How to start your lab and sign in to the Google Cloud console
	1. Click the Start Lab button. If you need to pay for the lab, a dialog opens for you to select your payment method. On the right is the Lab setup pane with the following:
		○ The Open Google Cloud console button
		○ The temporary credentials that you must use for this lab
		○ Other information, if needed, to step through this lab
	2. In the Lab Setup section, right-click Open Google Cloud console and select Open link in new window (or alternatively right-click and select Open Link in Incognito Window if you are running the Chrome browser).
The lab spins up resources, and then opens another tab that shows the Sign in page.
Tip: Arrange the tabs in separate windows, side-by-side.
Note: If you see the Choose an account dialog, click Use Another Account.
	3. If necessary, copy the Username below and paste it into the Sign in dialog.
student-01-6fa9fc9576d7@qwiklabs.net
Copied!

You can also find the Username in the Lab Setup pane.
	4. Click Next.
	5. Copy the Password below and paste it into the Welcome dialog.
JDqhAWSVOnKP
Copied!

You can also find the Password in the Lab Setup pane.
	6. Click Next.
Important: You must use the credentials the lab provides you. Do not use your Google Cloud account credentials.Note: Using your own Google Cloud account for this lab may incur extra charges.
	7. Click through the subsequent pages:
		○ Accept the terms and conditions.
		○ Do not add recovery options or two-factor authentication (because this is a temporary account).
		○ Do not sign up for free trials.
After a few moments, the Google Cloud console opens in this tab.
Note: To access Google Cloud products and services, click the Navigation menu or type the service or product name in the Search field. 

Task 1. Install ADK and set up your environment
In this task, you'll set up the Cloud Shell Terminal and Cloud Shell Editor to function as an IDE in Google Cloud. You'll then install Agent Development Kit (ADK) and download some sample code for the lab.
Note: Using an Incognito browser window is recommended for most Qwiklabs to avoid confusion between your Qwiklabs student account and other accounts logged into Google Cloud. If you are using Chrome, the easiest way to accomplish this is to close any Incognito windows, then right click on the Open Google Cloud console button at the top of this lab and select Open link in Incognito window.
Enable recommended Agent Platform APIs
	1. In this lab environment, the Agent Platform API has been enabled for you. If you were to follow these steps in your own project, you could enable it by navigating to Agent Platform and following the prompt to enable it.
Prepare a Cloud Shell Editor tab
	1. Click Activate Cloud Shell (

	) in the Google Cloud console title bar.
Note: Alternatively, select the console browser tab and press G then S to open the Cloud Shell terminal.
	2. Click Continue.
	3. When prompted to authorize Cloud Shell, click Authorize.
	4. In the upper right corner of the Cloud Shell Terminal panel, click the Open in new window button 

	.
	5. Click the Open Editor pencil icon (

	) at the top of the pane to view files.
	6. At the top of the left-hand navigation menu, click the Explorer icon (

	) to open your file explorer.
	7. Click the Open Folder button.
	8. In the Open Folder dialog that opens, click OK to select your Qwiklab student account's home folder.
	9. Close any additional tutorial or Gemini panels that appear on the right side of the screen to save more of your window for your code editor.
	10. In this lab, you will use the uv package manager developed by Astral. uv has become an industry standard to replace pip, venv, and other package and environment management tools. If you are unfamiliar with uv, read Astral's cheat-sheet for projects using uv or work through their series of guides to study it in greater depth.
Throughout the rest of this lab, you can work in this window as your IDE with the Cloud Shell Editor and Cloud Shell Terminal.
Install Terraform
Terraform is not pre-installed in Cloud Shell. You must install the Terraform CLI and configure it to persist across your Cloud Shell sessions.
	1. To install Terraform and ensure that the installation persists across sessions, run the following commands in the Cloud Shell terminal:
cat <<'EOF' > ~/.customize_environment
# Set up HashiCorp repository and install Terraform
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y terraform
EOF
bash ~/.customize_environment
Copied!
	2. To verify that Terraform has been installed correctly, run:
terraform --version
Copied!

The output should show the installed version (v1.5.7 or later):
Terraform v1.9.0
Install ADK and download code for this lab
	1. Paste the following commands into the Cloud Shell Terminal to copy the project directory containing the code for this lab from the Cloud Storage bucket:
gcloud storage cp -r gs://qwiklabs-gcp-04-84e68a312d15-bucket/adk_eval_challenge_lab .
Copied!
	2. Update your PATH environment variable, install project requirements, and initialize Terraform (used later to reset your data tables) by running the following commands in the Cloud Shell Terminal:
gcloud config set project qwiklabs-gcp-04-84e68a312d15
export PATH=$PATH:"/home/${USER}/.local/bin"
cd ~/adk_eval_challenge_lab
uv init
uv add -r requirements.txt
source .venv/bin/activate
terraform init
Copied!
	3. Write a .env file to provide authentication details for this agent directory by running the following in the Cloud Shell Terminal:
cat << EOF > bigquery_agent/.env
GOOGLE_GENAI_USE_VERTEXAI=TRUE
GOOGLE_CLOUD_PROJECT=qwiklabs-gcp-04-84e68a312d15
GOOGLE_CLOUD_LOCATION=global
MODEL=gemini-3.5-flash
EOF
Copied!
Task 2. Build and run the eval set
	1. In the Cloud Shell Editor, navigate to the file adk_eval_challenge_lab/bigquery_agent/evaluations/eval_config.json.
	2. Use the menus to toggle on Word Wrap with View > Word Wrap to make it easier to read and edit this JSON file.
	3. Notice that the config file specifies a metric, rubric_based_multi_turn_trajectory_quality_v1, which evaluates the actions that the agent takes leading to its response according to a rubric (or set of criteria) that you provide to it.
The file already contains one rubric to evaluate ledger_validity. Read the text_property for that criteria.
	4. Add a new rubric to the file, related to another issue that has been observed in the agent's behavior:
{
    "rubric_id": "valid_transitions",
    "rubric_content": {
        "text_property": "Valid transitions include: From pool_estimates to accepted_with_deposit or denied_estimates. From accepted_with_deposit to scheduled_installations. From scheduled_installations to completed_pools. From completed_pools to paid_and_closed."
    }
}
Copied!
	5. Run the following check in the Cloud Shell Terminal to confirm your JSON is valid after the addition in the previous step:
if python3 -m json.tool bigquery_agent/evaluations/eval_config.json > /dev/null 2>&1; then echo "Valid"; else echo "Invalid"; fi
Copied!
	6. Use the adk eval_set create command to create a new evaluation base configuration named ledger. This will output a file named bigquery_agent/ledger.evalset.json.
	7. Inspect the new bigquery_agent/ledger.evalset.json and read the scenarios in bigquery_agent/evaluations/scenarios.json. You may want to enable View > Word Wrap on the files for easier reading.
	8. Use the adk eval_set add_eval_case to add the ledger you just created. Make sure to add the scenarios file called bigquery_agent/evaluations/scenarios.json to the evalset, and the session input file bigquery_agent/evaluations/session_input.json.
	9. Run evaluations using ADK. Pipe the results to tee to both display the results in the console and save them to a text file so you can process the resulting tables to make them easier to interpret:
adk eval bigquery_agent ledger \
--config_file_path bigquery_agent/evaluations/eval_config.json \
--print_detailed_results \
--log_level=CRITICAL \
| tee eval_results.txt
Copied!
	10. Run the following command to display fewer columns from the output to make it easier to read:
cat eval_results.txt | cut -d'|' -f3,4,7,8 | sed -E 's/^.*[-=]{3,}.*$//; /^[| ]+$/d'
Copied!

Expected results:
For each of the evaluation scenarios, a table will be generated with the results and evaluation rationale for the result for each rubric. Above each table, you will see a summary.
From the tables or the summaries, you can identify whether each rubric passed or failed for each of the cases. In the tables, the score for that rubric is posted at the bottom of the cell containing the reasoning for that rubric, with a 1.0 representing a PASS and a 0.0 representing a FAIL.
The expected results for this evaluation are:
	Scenario	Rubric: ledger_validity	Rubric: valid_transitions
	Clark Kent	PASSED: adds a record and deletes a record	FAILED: transaction goes from scheduled_installations -> paid_and_close instead of completed_pools first.
	Ron Weasley	FAILED: deletes a record without adding it to another table	PASSED: does not perform an addition to the wrong table
	Bob Jones	PASSED: adds a record and deletes a record	PASSED: pool_estimates -> accepted_with_deposit is a valid transition
Reset the Lab's Tables
	1. After running the evaluation, reset the tables used in order to prepare for your next iteration of the evaluation by running the following commands:
cd ~/adk_eval_challenge_lab
terraform apply -var="gcp_project_id=qwiklabs-gcp-04-84e68a312d15" -auto-approve
Copied!

Click Check my progress to verify the objective.
Build and run the evaluation set
Task 3. Improve the agent to fix evaluation issues
In this task, you will improve the agent and then run the evals again to see the results. Specifically, you will modify the agent to use more specific tools instead of the open-ended BigQuery Toolset.
	1. In the Cloud Shell Editor, open the file bigquery_agent/agent.py.
	2. Complete the function named perform_consistent_transaction to read from a table, add a record, and then delete a record all in one transaction. Notice that there are SQL helper functions that do these tasks independently, which you can combine to form your new function.
	3. Complete the function named check_transaction to check if a transition between two tables is valid and return True if so. The valid transitions are listed here:
- "pool_estimates" -> "accepted_with_deposit"
- "pool_estimates" -> "denied_estimates"
- "accepted_with_deposit" -> "scheduled_installations"
- "scheduled_installations" -> "completed_pools"
- "completed_pools" -> "paid_and_closed"
Copied!
	4. Modify the agent so that it only uses the following tools:
		○ read_table
		○ read_table_all
		○ check_transaction
		○ perform_consistent_transaction
	5. Replace the last line of the agent's instruction (which currently reads "Query all available tables") with the following to provide the agent more information about the available tables and when to use the tools you have provided to it. Make sure you remove all mentions of the bigquery_toolset.
The tables you have available are:
  - pool_estimates: Contains all pool estimates
  - accepted_with_deposit: Contains all pool estimates that have been accepted and have a deposit
  - denied_estimates: Estimates that have been denied by the customer and will not proceed.
  - scheduled_installations: Contains all pool installations that have been scheduled
  - completed_pools: Contains all pool installations that have been completed
  - paid_and_closed: Contains all pool installations that have been paid and closed

Use read_table_all to read the data from the tables.
Use check_transaction to check if a transaction is valid before performing any transactions. If not valid, tell the user so.
Use perform_consistent_transaction when you need to read a table, insert a row into another table and delete the original row.
Copied!
	6. Run the evaluations again using the ADK CLI, again using tee to save the results to a file:
adk eval bigquery_agent ledger \
--config_file_path bigquery_agent/evaluations/eval_config.json \
--print_detailed_results \
--log_level=CRITICAL \
| tee improved_eval_results.txt
Copied!

	7. Run the following command to display fewer columns from the output to make it easier to read:
cat improved_eval_results.txt | cut -d'|' -f3,4,7,8 | sed -E 's/^.*[-=]{3,}.*$//; /^[| ]+$/d'
Copied!

You should see the following details with all the tests working now:
	Scenario	Rubric: ledger_validity	Rubric: valid_transitions
	Clark Kent	PASSED: it does not add a record	PASSED: it does not add a record
	Ron Weasley	PASSED: it does not delete a record	PASSED: does not perform an addition to the wrong table
	Bob Jones	PASSED: adds a record and deletes a record	PASSED: pool_estimates -> accepted_with_deposit is a valid transition.
	8. Upload the agent to Cloud Storage for the lab autograder to evaluate your updates.
gcloud storage cp -r ./bigquery_agent/agent.py gs://qwiklabs-gcp-04-84e68a312d15-bucket/
Copied!
Click Check my progress to verify the objective.
Improve your agent and upload it to Cloud Storage

Congratulations!
In this lab, you've learned how to evaluate and improve Agent Development Kit (ADK) agents. You set up the ADK environment and built custom evaluation sets with rubrics and scenarios. Then, you ran evaluations to identify agent issues. You fixed these issues by implementing data consistency and transition validation tools. Finally, you verified the fixes and uploaded the agent to Cloud Storage.
End your lab
When you have completed your lab, click End Lab. Qwiklabs removes the resources you’ve used and cleans the account for you.