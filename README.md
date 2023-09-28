#Lambda and S3Events Demonstration

This project aims to showcase a simple event-driven image processing pipeline using AWS Lambda and S3 buckets. The pipeline comprises two buckets: a source bucket and a processed bucket. Whenever new images are added to the source bucket, a Lambda function is triggered based on the PUT operation. Upon invocation, the Lambda function receives the event and extracts the bucket and object information. Utilizing the `PIL` module, the function pixelates the image with five different variations (8x8, 16x16, 32x32, 48x48, and 64x64) and uploads them to the processed bucket.

## Stage 1 - Creating the S3 Buckets

To begin, navigate to the S3 Console at [https://s3.console.aws.amazon.com/s3/home?region=us-east-1#](https://s3.console.aws.amazon.com/s3/home?region=us-east-1#).

Two buckets will be created, each with the same name suffixed by a functional title (as indicated below). Leave all settings at their defaults, except for the region and bucket name.

1. Click on "Create Bucket" and create a bucket with the format `unique-name-source` in the `us-east-1` region.
2. Click on "Create Bucket" again and create another bucket with the format `unique-name-processed` in the `us-east-1` region.

Ensure that the bucket names are unique. For example:

- Bucket 1: `dontusethisname-source`
- Bucket 2: `dontusethisname-processed`

	![Untitled](/images/Untitled.png)

## Stage 2 - Creating the Lambda Role

Next, proceed to the IAM Console at [https://console.aws.amazon.com/iamv2/home?#/home](https://console.aws.amazon.com/iamv2/home?#/home).

1. Click on "Roles" and then "Create Role."
2. Select "AWS service" for the "Trusted entity type" field.
3. Choose "Lambda" as the service to trust, and click "Next" twice.
4. Provide the "Role name" as `PixelatorRole` and create the role.
	![Untitled](/images/Untitled1.png)
5.	Click on the newly created role `PixelatorRole`.

To add permissions, an inline policy must be created under "Permissions Policy."
	![Untitled](/images/Untitled2.png)

1. Click on "JSON" and delete the existing contents within the code box.
2. Open this [link](https://raw.githubusercontent.com/Gbengard/aws-lambda-s3-events/main/Dependencies/policy/s3pixelator.json) in a new tab.
3. Copy the entire contents of the page and paste them into the code editor box for the permissions policy.
4. Locate the word `REPLACEME` (four occurrences in total), representing the bucket and object names for both the source and processed buckets.
	![Untitled](/images/Untitled3.png)
5. Replace each occurrence of `REPLACEME` with the respective name you selected for your buckets. For example, in my case, it is `dontusethisname`.
6. Locate the two occurrences of `YOURACCOUNTID` and replace both with your AWS account ID. To obtain your account ID, click on the account dropdown at the top right corner, copy the "Account ID" (removing the `-` if present), and replace `YOURACCOUNTID` in the policy code editor. Ensure you paste the account ID as `123456789000` rather than `1234-5678-9000`.

The modified policy code should resemble the following, with your own account ID:

```
{
	"Effect": "Allow",
	"Action": "logs:CreateLogGroup",
	"Resource": "arn:aws:logs:us-east-1:123456789000:*"
},
{
	"Effect": "Allow",
	"Action": [
		"logs:CreateLogStream",
		"logs:PutLogEvents"
	],
	"Resource": [
		"arn:aws:logs:us-east-1:123456789000:log-group:/aws/lambda/pixelator:*"
	]
}
```

7.	Click on "Next" and name the policy `pixelator_access_inline`. Finally, create the policy.

# Stage 3 (pre) - Creating a Lambda Zip (Optional)

**Note: This guide has been tested on macOS and should work on Linux. However, Windows may require different tools. If unsure, please proceed to step 3 below.**

To create a Lambda deployment package, follow these steps from the CLI/Terminal:

1. Create a folder named `my_lambda_deployment`.
2. Navigate into the `my_lambda_deployment` folder.
3. Create a subfolder called `lambda`.
4. Move into the `lambda` folder.
5. Create a file named `lambda_function.py` and copy the code for the lambda `pixelator` function from [this link](https://raw.githubusercontent.com/Gbengard/aws-lambda-s3-events/main/Dependencies/lambda/lambda_function.py) into the file. Save the file.
6. Download the file from [this link](https://files.pythonhosted.org/packages/f3/3b/d7bb231b3bc1414252e77463dc63554c1aeccffe0798524467aca7bad089/Pillow-9.0.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl) and place it in the `lambda` folder.
7. Run the following commands in the same folder:
   - `unzip Pillow-9.0.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl`
   - `rm Pillow-9.0.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl`
   These commands extract the necessary files for the Pillow module, which is required for image manipulation in Python 3.9 (the version used by the lambda function).
8. From the same folder, execute the command `zip -r ../my-deployment-package.zip .`. This command will create a zip file named `my-deployment-package.zip` in the parent directory, containing all the required files.
	![Untitled](/images/Untitled4.png)

The resulting zip file will be the same as the one provided in the link below. If you encounter any issues with the lambda function, you can use the pre-created zip file.

# Stage 3 - Create the Lambda Function

Follow these steps to create the Lambda function:

1. Go to the Lambda console by accessing [this link](https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions).
2. Click on the "Create Function" button.
3. Choose the option "Author from scratch".
4. Enter the function name as "pixelator".
5. Select "Python 3.9" as the runtime.
6. Choose "x86_64" as the architecture.
7. Expand the "Permissions" section and click on "Change default execution role".
8. Select "Use an existing role" and choose "PixelatorRole" from the "Existing role" dropdown.
	![Untitled](/images/Untitled5.png)
9. Click on the "Create Function" button.
10. Close any notification dialogues or popups that appear.
11. Click on "Upload from" and select the option to upload a `.zip` file.
	![Untitled](/images/Untitled6.png)
    - You have two options:
      - Download the zip file to your local machine from [this link](https://github.com/Gbengard/aws-lambda-s3-events/blob/main/Dependencies/my-deployment-package.zip) and then upload it.
      - Use the zip file created in the previous "Stage 3 (pre)" step. Both zip files are identical.
12. On the Lambda screen, click on "Upload", locate and select the zip file, and then click the "Save" button.
	![Untitled](/images/Untitled7.png)
    - Please note that the upload process may take a few minutes. Once completed, you might see a message stating that the deployment package of your Lambda function "pixelator" is too large to enable inline code editing. However, you can still invoke your function, which is acceptable.

# Stage 4 - Configure the Lambda Function & Trigger

Configure the Lambda function and trigger as follows:

1. Click on the "Configuration" tab, and then select "Environment variables".
2. Add an environment variable that specifies the processed bucket for the pixelator function. The source bucket is automatically provided through the event data.
3. Click on "Edit" and then "Add environment variable".
4. Set the "Key" as "processed_bucket" and enter the name of *your* processed bucket as the "Value".
   - For example: `dontusethisname-processed` (replace it with *your* bucket name).
   - Be absolutely sure to specify your *processed* bucket and **not** your source bucket.
		![Untitled](/images/Untitled8.png)
     - Using the source bucket here will cause the output images to be stored in the source bucket, triggering the lambda function repeatedly.
     - Ensure to specify your processed bucket correctly.
5. Click "Save" to apply the changes.

6. Click on "General configuration", then click on "Edit" and set the timeout to "1" minute and "0" seconds.
7. Click "Save" to apply the changes.

8. Click on "Add trigger".
	![Untitled](/images/Untitled9.png)
9. Select "S3" from the dropdown menu.
10. Choose your *source* bucket under the "Bucket" section.
    - Once again, double-check that you select the correct *source* bucket and **not** the destination or any other bucket.
	![Untitled](/images/Untitled10.png)
11. Acknowledge the "Recursive invocation" checkbox. This lambda function is triggered whenever anything is added to the *source* bucket.
    - Configuring this incorrectly or providing the wrong environment variable may result in the lambda function running indefinitely.
12. Click "Add" to create the trigger.
	![Untitled](/images/Untitled11.png)

## Stage 5 - Testing and Monitoring

Perform the following steps to test and monitor the GitHub project:

1. Open a new tab in your browser and navigate to the CloudWatch Logs console using the following URL: [https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logsV2:log-groups](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logsV2:log-groups).
2. Ensure that you have two tabs open for the S3 console. In one tab, open the `-source` bucket, and in the other tab, open the `-processed` bucket.
3. In the `-source` bucket tab, select the "Objects" tab and click on "Upload" to upload some files. You can use your own files or obtain them from [this location](https://github.com/Gbengard/aws-lambda-s3-events/tree/main/Dependencies/media).
4. After uploading the files, click "Close" to exit the upload dialog.
5. Switch to the CloudWatch Logs tab.
6. Click the "Refresh" icon in the console and locate the `/aws/lambda/pixelator` log stream. If available, click on the most recent log stream. If it's not visible, continue clicking the "Refresh" icon until the most recent log stream appears.
	![Untitled](/images/Untitled12.png)
7. Expand the line starting with `{'Records': [{'eventVersion':` to view detailed information about the lambda invocation event. The object name should be listed under `'object': {'key'`.
8. Now, go to the S3 Console tab for the `-processed` bucket.
9. Click the "Refresh" icon in the console to update the view.
	![Untitled](/images/Untitled13.png)
10. Select each of the pixelated versions of the image, which should include `8x8`, `16x16`, `32x32`, `48x48`, and `64x64` versions.
11. Click "Open" to view the images. Your browser will either display the images or prompt you to save them.
12. Open each image one by one, starting with `8x8` and ending with `64x64`. Notice how the images become less pixelated as you progress.

## Stage 6 - Cleanup

Follow the steps below to clean up the project:

1. Open the `pixelator` lambda function in your browser using the following URL: [https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions/pixelator?tab=code](https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions/pixelator?tab=code).
	![Untitled](/images/Untitled14.png)
2. Delete the lambda function from the console.
3. Proceed to the IAM Roles console by visiting [https://console.aws.amazon.com/iamv2/home#/roles](https://console.aws.amazon.com/iamv2/home#/roles).
4. Select the `PixelatorRole`, click on "Delete," and confirm the deletion.
	![Untitled](/images/Untitled15.png)
5. Go back to the S3 Console using the following URL: [https://s3.console.aws.amazon.com/s3/home?region=us-east-1&region=us-east-1](https://s3.console.aws.amazon.com/s3/home?region=us-east-1&region=us-east-1).
6. For each of the `source` and `processed` buckets, follow these steps:
   - Select the bucket.
   - Click on "Empty."
   - Type "permanently delete" in the dialog box and confirm the action.
   - Close the dialogue and return to the main S3 Console.
   - Ensure the bucket is still selected and click on "Delete."
   - Type the name of the bucket and delete it.
7. Go to CloudWatch console by visiting [https://us-east-1.console.aws.amazon.com/cloudwatch/home?region=us-east-1#logsV2:log-groups](https://us-east-1.console.aws.amazon.com/cloudwatch/home?region=us-east-1#logsV2:log-groups)
   - Check the "/aws/lambda/pixelator" and click on "Actions"
   - Click on "Delete log group(s)"
   ![Untitled](/images/Untitled16.png)

That concludes the testing, monitoring, and cleanup process for the project.
