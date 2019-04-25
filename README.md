# custom-resource-s3-bucket-delete

## Summary
This CloudFormation template defines a Lambda-backed custom resource to handle S3 bucket deletes. 

## Description

If you have a CloudFormation template that creates an S3 bucket, you might face issues when deleting the stack if the bucket is not empty.  This is documented in the ['AWS::S3::Bucket' resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html):

- `Important: You can only delete empty buckets. Deletion fails for buckets that have contents`

This CloudFormation template defines a [Lambda-backed custom resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources-lambda.html) that can be used to fix this.  The custom resource will do nothing but return a `success` message to CloudFormation during a create-stack and update-stack operation.  However, during the delete-stack operation, it will programatically empty the S3 bucket.


Also, since you will need to pass the name of the S3 bucket to the custom resource using the [Fn::Ref](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html) intrinsic function, this creates an implicit dependency on these two resources that ensures that the custom resource will be deleted first (where it empties the bucket) and then the S3 bucket is deleted second.  Because of this, we can be sure that the S3 bucket will be empty when CloudFormation starts to delete the resource.


## Use

This CloudFormation template contains the following:

- Sample S3 bucket for testing
- custom resource
- Lambda function that is invoked by custom resource
- Lambda function execution role

You can use this resource in one of two ways:

1) Test the template directly:

    - Test the functionality by using the template to create a stack with the following command:

        `aws cloudformation create-stack --stack-name custom-resource --template-body file://custom-resource.yml --capabilities CAPABILITY_IAM`

    - Once the stack is created, add any number of objects to the S3 bucket
    - After this, delete the stack, which should complete successfully.  You can verify the order of deletion in the stack events, verifying that the custom resource is deleted first, then the S3 bucket.  You can also see the output of the Lambda function in CloudWatch Logs

2) Use the custom resource in your own template:

    - you can add everything in this template to your existing template with the following notes:

        - You can remove the 'S3Bucket' resource in the template, as it is just used for testing
        - You need to edit two items in the 'CustomResource' resource:

            - Change the `DependsOn` property to the logical name of the S3 bucket in your template
            - change the parameter passed to the custom resource to reference your S3 bucket.  So you should change this line:

                `s3bucketName: !Ref S3Bucket`

            - to this:

                `s3bucketName: !Ref <your s3 bucket logical ID>`

