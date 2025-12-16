# state-mv

I understand. You need a clear, start-to-finish summary of the steps we took to migrate your existing S3 bucket from a standalone Terraform resource to being managed by the new module, including the specific configuration fixes we identified.

Here are the complete, consolidated steps for your documentation:

📄 Documentation: S3 Resource Migration to Module
Context
Existing Bucket Name: test-bucket-module-a2d3c75f

Old Resource Address (Source): aws_s3_bucket.test_bucket_module

New Module Address (Destination): module.test_bucket_module.module.this.aws_s3_bucket.this[0]

Module Source: github.com/hinge-health-terraform/aws_s3_bucket?ref=v3.6.1

Problem Encountered: The module was automatically prepending an unwanted prefix (e.g., hingehealth-) to the bucket name, requiring an override.

Fix Identified: Using the name_prefix = "" argument in the module call.

🛠️ Phase 1: Configuration Refactoring
The goal is to remove the old resource block and introduce the new module block, ensuring the bucket name remains exactly the same.

Step 1: Remove the Old Resource Block
Completely delete the original, standalone resource block from your Terraform configuration file:

Terraform

# DELETE THIS BLOCK
resource "aws_s3_bucket" "test_bucket_module" {
  bucket = "test-bucket-module-${random_id.this.hex}"
  provider = aws.us-east-1
  tags = module.label.tags
}
Step 2: Add the New Module Block (With Fixes)
Add the new module block. This configuration uses the hardcoded bucket name to ensure an exact match for the state move and includes the name_prefix = "" fix.

Terraform

# ADD THIS BLOCK
module "test_bucket_module" {
  source = "github.com/hinge-health-terraform/aws_s3_bucket?ref=v3.6.1"

  # CRITICAL: Use the exact existing bucket name (Hardcoded)
  name             = "test-bucket-module-a2d3c75f" 
  
  # CRITICAL FIX: Override the module's default prefix (e.g., 'hingehealth-')
  # This stops the module from trying to rename the bucket.
  name_prefix      = ""

  providers = {
    aws    = aws.us-east-1
    aws.dr = aws.us-east-2
  }
  
  context          = module.label.context
  force_destroy    = false
  object_ownership = "BucketOwnerEnforced"
  private          = true
  tags             = module.label.tags
}
Step 3: Run Initial Plan Check
Run terraform plan to verify the state migration is required.

Bash

terraform plan
Expected Plan Output: The plan should show 1 resource to destroy (aws_s3_bucket.test_bucket_module) and 1 resource to create (module.test_bucket_module.module.this.aws_s3_bucket.this[0]). (If it showed replacement, it meant the prefix fix was not yet implemented or wrong, but with the fix, this is the expected intermediate state).

🔗 Phase 2: State Migration
The goal is to instruct Terraform that the existing bucket is now managed by the new module path.

Step 4: Execute the State Move Command
Use the terraform state mv command to migrate the S3 bucket from the old resource address to the new module address.

Bash

terraform state mv 'aws_s3_bucket.test_bucket_module' 'module.test_bucket_module.module.this.aws_s3_bucket.this[0]'
Verification: The command should output: Successfully moved 1 object(s).

🚀 Phase 3: Final Verification and Apply
The goal is to confirm the migration worked and apply the new module defaults (like Versioning, Public Access Block, etc.).

Step 5: Run Final Plan Check
Run terraform plan to confirm the move was successful and no destructive changes are planned.

Bash

terraform plan
Expected Plan Output: The plan should show:

NO destruction on the S3 bucket itself.

1 resource to change (~ update in-place) for the S3 bucket (to apply any new tags).

5+ resources to add (+ create) for module helper resources (e.g., aws_s3_bucket_public_access_block, aws_s3_bucket_versioning).

Step 6: Apply Changes
Apply the plan to finalize the migration and integrate the module's standard security and lifecycle configurations.

Bash

terraform apply --auto-approve
The bucket is now successfully managed by the module and ready for further configuration (like enabling DR).
