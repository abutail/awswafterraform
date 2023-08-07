# awswafterraform
terraform webacl
1. Modularity: Organize your Terraform code into reusable modules for better maintainability and readability. This approach allows you to create specific configurations for different use cases.

2. Version Control: Use version control systems like Git to track changes in your Terraform code, enabling collaboration with other team members and easy rollbacks if needed.

3. Parameterization: Utilize variables and input parameters to make your code flexible and adaptable to different environments or configurations.

4. Resource Naming: Use meaningful and consistent names for your resources to enhance readability and clarity.

5. Resource Tagging: Implement resource tagging to categorize and manage your AWS WAF resources effectively.

6. Terraform Backend: Consider using remote backends like AWS S3 or Terraform Cloud to store your state files securely and enable better collaboration in a team environment.

7. Terraform Modules: Leverage community-contributed Terraform modules to simplify the implementation of AWS WAF rules and conditions.

8. Plan and Apply Separation: Run "terraform plan" before applying changes to ensure that you understand the changes being made to your infrastructure.

9. Version Locking: Lock Terraform provider versions to ensure consistency and avoid unexpected issues when new versions are released.

10. Documentation: Write clear comments and documentation for your Terraform code to aid understanding and future maintenance.

Remember, best practices may evolve over time, so always stay updated with the latest Terraform and AWS WAF features and recommendations.

provider "aws" {
  region = "us-west-2" # Replace with your desired AWS region
}

# Define the AWS WAF WebACL
resource "aws_wafv2_web_acl" "example_web_acl" {
  name        = "ExampleWebACL"
  description = "Example WebACL for AWS WAF"

  scope = "REGIONAL" # Use "CLOUDFRONT" for CloudFront distributions

  default_action {
    allow {} # Change to "block {}" if you want to block by default
  }

  # Define a rule group for common rule settings
  rule_group_reference_statement {
    name = "AWSManagedRulesCommonRuleSet"
  }

  # Define custom rules
  rule {
    name     = "ExampleRule1"
    priority = 10

    statement {
      byte_match_statement {
        field_to_match {
          single_header {
            name = "User-Agent"
          }
        }
        positional_constraint = "CONTAINS"
        search_string         = "badbot"
        text_transformation   = "LOWERCASE"
      }
    }

    action {
      block {}
    }
  }

  rule {
    name     = "ExampleRule2"
    priority = 20

    statement {
      rate_based_statement {
        limit            = 200
        aggregate_key_type = "IP"
        scope_down_statement {
          regex_pattern_set_reference_statement {
            arn = aws_wafv2_regex_pattern_set.example_regex_set.arn
          }
        }
      }
    }

    action {
      block {}
    }
  }
}

# Define a custom regex pattern set for ExampleRule2
resource "aws_wafv2_regex_pattern_set" "example_regex_set" {
  name        = "ExampleRegexSet"
  description = "Example Regex Pattern Set for AWS WAF"

  scope = "REGIONAL" # Use "CLOUDFRONT" for CloudFront distributions

  regular_expression {
    regex_string = "example-pattern"
  }
}

In this sample code, we’re creating an AWS WAF WebACL named “ExampleWebACL.” It includes two rules: “ExampleRule1,” which blocks requests with a User-Agent header containing “badbot,” and “ExampleRule2,” which blocks IP addresses if they exceed a specified rate limit and match a custom regex pattern.

Please ensure you have appropriate AWS credentials and permissions set up to execute this code successfully. Remember to adapt it to your specific use case and modify any values or settings as needed.
