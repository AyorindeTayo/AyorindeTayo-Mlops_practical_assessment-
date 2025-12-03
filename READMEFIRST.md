# MLE Assessment — Hosted DB Edition Overview
- Train XGBoost on data/features_training.csv (see notebooks/train_credit_risk_model.ipynb).
- Connect to hosted PostgreSQL (see data/Remote Database Credentials updated.docx).
-   Note that the relevant tables in the DB are: `bank_transactions` + `bank_transactions_scoring`
- Engineer features from bank_transactions_scoring per feature_spec.md and score with your model.
- Use starter/ for Lambda + Docker packaging.


## Assessment rule: No prebuilt views
- We intentionally do **not** provide a flattened view.
- Candidates must expand the nested `transactions` JSONB via SQL (lateral `jsonb_array_elements`) and compute features themselves.
- Please **do not** create server-side views or tables; use read-only SELECTs/CTEs in your submission.


## Submission template
Use `SUBMISSION_TEMPLATE.md` to structure your deliverables and reproduction steps.



# Instructions:

You have been provided with a jupyter notebook (located in /notebooks). This notebook is a simple way of training a model and making a prediction. 

## The first task:
	Produce a model file that can be imported by libraries such as joblib
	This model file should allow you to load the model in a completely separate script which is independent of the training notebook

## The second task:
	Produce a python script in the AWS Lambda format
	This script must load the model file, accept an input event and produce a response containing the prediction the model has made.
	The script should receive a single JSON body for a singular prediction and return the prediction for the data given.

## The third task (optional extra)
	Make use of the lambda-local python library to test your lambda script
	
## The fourth task:
	Produce a docker image containing the script and required libraries required to execute task 2.
	Run the docker image

## The fifth task: (part of assessment 2)
	Make a call with curl/Postman to the docker image, supplying it with input parameters (JSON body containing the required parameters for a single inference)
	This should return the prediction in the format from task 2.
	You should retrieve the data to send to the model from .sql file supplied (save it as a JSON file for easy reference)
	The final file structure for this task should contain the Dockerfile, model and lambda python script.


## Final thoughts:
	The idea behidn the assessment is to recreate a generalised model, that would generally score a batch of data, into a model or script that receives a single set of input parameters, which closer simulates production environments