# AWS Config Resource Counter - Implementation Specification

## Overview

Create two scripts to scan AWS Config for resource counts across all regions and resource types in an AWS account. Output results to CSV with comprehensive error handling.

## Prerequisites

- macOS with bash 3.2+ compatibility
- pyenv installed and available in PATH
- AWS credentials configured (via ~/.aws/credentials or environment variables)
- AWS Config enabled in at least one region
- IAM permissions: config:ListDiscoveredResources, config:ListResourceCounts, sts:GetCallerIdentity, iam:ListAccountAliases, ec2:DescribeRegions

## Script 1: aws_config_setup.sh

### Purpose

Bash script to setup pyenv environment and execute Python script autonomously.

### Requirements

#### 1. Bash 3.2 Compatibility

Must work with default macOS bash version 3.2

#### 2. Check pyenv Installation

- Verify pyenv command available in PATH
- Exit with error message if not found
- Provide installation URL if missing
- This is the only prerequisite that requires human intervention

#### 3. Automatic Python 3.11 Setup

- Check if any Python 3.11.x version is installed via pyenv versions command
- If no Python 3.11.x found:
  - Automatically install latest Python 3.11 using pyenv install 3.11
  - Display installation progress to user
  - Exit with error if installation fails
- If Python 3.11.x is installed:
  - Identify the latest 3.11.x version available
  - Run pyenv local with that version to activate in current directory
  - This creates or updates .python-version file
- After setup, verify active Python version using pyenv version command
- Confirm active version matches expected 3.11.x
- Exit with error if verification fails

#### 4. Install Required Packages

- Use pip install --quiet boto3 aioboto3
- Always run installation (pip skips if already satisfied)
- Check exit status and fail with error if installation fails

#### 5. Execute Python Script

- Run python get_config_counts.py
- Start timer before execution begins
- Pass through Python script exit code
- Calculate total execution time when complete
- Display execution time in seconds with format: "Total execution time: X seconds"

#### 6. Error Handling

- Each step must verify success before proceeding
- Display clear error messages with actionable guidance
- Exit with non-zero code on any failure
- Never proceed to next step if current step fails
- Only pyenv installation requires human intervention
- All other failures should provide clear error context

### Output Format

Display progress messages for each major step:

- Checking pyenv installation
- Setting up Python 3.11 environment (with installation if needed)
- Installing required packages
- Running AWS Config resource counter
- Blank line before execution time
- Total execution time at completion

## Script 2: get_config_counts.py

### Purpose

Python script to enumerate AWS resources via Config API and output counts to CSV.

### Python Version Target

Python 3.11

### Code Standards

- Follow Black formatter standards (88 character line length)
- Single script file
- All code wrapped in try-except blocks
- Type hints where appropriate for function signatures

### Prerequisites Verification Function

Must execute as first operation before main execution begins.

#### Prerequisites to Check (in sequential order)

1. boto3 package installed and importable
2. aioboto3 package installed and importable
3. AWS credentials configured (test via boto3.Session)
4. IAM permission: sts:GetCallerIdentity
5. IAM permission: iam:ListAccountAliases
6. IAM permission: ec2:DescribeRegions
7. IAM permission: config:ListDiscoveredResources (test in us-east-1)
8. AWS Config enabled in at least one region (test us-east-1, us-west-2, eu-west-1)

#### Prerequisite Check Output Format

Display header: "Verifying prerequisites..."

For each prerequisite:

- Green check (✓) with description if passed
- Red X (✗) with description if failed
- Use ANSI color codes: \033[92m for green, \033[91m for red, \033[0m to reset

#### Prerequisite Check Behavior

- Execute ALL prerequisite checks sequentially
- Do not stop at first failure
- Collect all failures as checks progress
- After all checks complete:
  - If any failures exist:
    - Display separator line
    - Display "PREREQUISITE FAILURES:" header
    - List each failure with details
    - Write failures to error file
    - Exit with code 1
  - If all pass:
    - Proceed to main execution

### Main Execution Logic

#### Step 1: Get Account Information

Use STS GetCallerIdentity to retrieve account ID
Use IAM ListAccountAliases to retrieve account alias (if exists)
Store both for later use in filenames and CSV output

#### Step 2: Setup Error File

Create error filename based on account info:

- With alias: aws_config_resource_count_{accountId}_{accountAlias}_errors.txt
- Without alias: aws_config_resource_count_{accountId}_errors.txt
- Delete error file if it already exists from previous run

#### Step 3: Get All Enabled Regions

Use EC2 DescribeRegions API in us-east-1
Extract RegionName from each region in response
Return sorted list of region names

#### Step 4: Parallel Region Scanning

Use asyncio with aioboto3 to scan regions concurrently

For each region:

- Create async Config client with retry configuration
- Use Standard retry mode with max_attempts=10
- Query for common AWS resource types
- Resource types to check: AWS::EC2::Instance, AWS::EC2::Volume, AWS::EC2::SecurityGroup, AWS::EC2::VPC, AWS::S3::Bucket, AWS::RDS::DBInstance, AWS::Lambda::Function, AWS::IAM::Role, AWS::IAM::User, AWS::DynamoDB::Table
- Use list_discovered_resources with pagination
- Count total resources per resource type
- Only include resource types with count greater than zero
- Handle errors per resource type and continue

Progress indicator:

- Display "Scanning N regions..." at start
- Show progress bar updated as each region completes
- Format: Progress: [████████░░░░] completed/total
- Use 50 character width progress bar
- Print newline after progress completes

Error handling during scan:

- TooManyRequestsException: Log, sleep 1 second, continue
- NoSuchConfigurationRecorderException: Log Config not enabled, skip region
- AccessDeniedException: Log permission denied, skip region/resource
- OptInRequired: Log region not enabled, skip region
- ResourceNotDiscoveredException: Skip resource type silently
- All other ClientError: Log and continue
- All other Exception: Log and continue

Never allow single region or resource failure to stop entire scan

#### Step 5: Output to CSV

CSV Filename format:

- With alias: aws_config_resource_count_{accountId}_{accountAlias}.csv
- Without alias: aws_config_resource_count_{accountId}.csv

CSV Columns (header row required):

- account
- alias
- region
- resource_type
- total

CSV Content requirements:

- One row per region per resource type combination
- account column: AWS account ID as string
- alias column: Account alias or empty string if none
- region column: AWS region code
- resource_type column: AWS Config resource type identifier
- total column: Integer count of resources
- Only include rows where total is greater than zero
- Sort rows by region ascending, then resource_type ascending

CSV Writing approach:

- Write to temporary file first using tempfile.mkstemp
- Use csv.DictWriter with specified field order
- Write header row first
- Write all data rows
- Use os.replace for atomic rename to final filename
- Clean up temporary file on any error

### Error Logging Requirements

#### Error File Format

Filename pattern already described in Step 2 above

Error log line format:

- [ISO8601 timestamp] [ERROR_TYPE] Region: {region}, Resource: {type}, Message: {msg}
- Region and Resource fields optional depending on error context
- Include all three when available
- Timestamp in UTC using ISO format

#### Error Types to Log

- PREREQUISITE: Failed prerequisite check
- THROTTLE: Rate limiting encountered
- PERMISSION: Access denied errors
- REGION_DISABLED: Region not enabled for account
- CONFIG_DISABLED: AWS Config not enabled in region
- API_ERROR: AWS API returned error
- CONNECTION: Network or connection errors
- SCAN_FAILED: Region scan failed completely
- FATAL: Unrecoverable error
- UNEXPECTED: Catch-all for unexpected exceptions

#### Error Output Destinations

Write to both:

1. Console stderr
2. Error file in append mode

Always attempt both even if one fails

#### Error Handling Principles

- Never crash script due to single region failure
- Never crash script due to single resource type failure
- Always complete CSV output with successfully collected data
- Graceful degradation: collect what is accessible, log what is not
- Provide actionable error messages with context

### Script Structure

Main function flow:

1. Verify prerequisites (exit if any fail)
2. Get account info (exit on failure)
3. Get all regions (exit on failure)
4. Scan regions asynchronously (continue with partial results)
5. Write CSV atomically (exit on failure)
6. Display summary statistics

Top-level exception handlers:

- KeyboardInterrupt: Print interrupted message, exit 130
- All other Exception: Print fatal error, exit 1

Module structure:

- Shebang line for python3
- Module docstring
- Import statements with try-except for boto3 and aioboto3
- Constants (ANSI color codes)
- Helper functions (log_error, verify_prerequisites, get_account_info, get_all_regions)
- Async functions (scan_region, scan_all_regions)
- Data output function (write_csv)
- Main function coordinating execution
- if __name__ == "__main__" guard with top-level exception handling

### Performance Considerations

Async execution:

- Scan all regions in parallel using asyncio.gather or asyncio.as_completed
- Respect per-region API rate limits (separate quota per region)
- Each region scanned independently

Pagination:

- Use aioboto3 paginator for list_discovered_resources
- Handle large result sets without loading all into memory at once

Retry logic:

- Use boto3 Standard retry mode with exponential backoff
- Maximum 10 retry attempts per request
- Handles transient failures automatically

Memory management:

- Do not hold all results in memory before writing
- Collect results incrementally as regions complete
- Write to CSV after all scans complete (results are just counts, not large)

### Output Example

Console output flow:

1. Prerequisite checks with check marks or X marks
2. Account scanning message with account ID and alias if present
3. Regions found count message
4. Progress bar during scanning
5. Results written to filename message
6. Errors logged to filename message
7. Summary section with:
   - Regions scanned count
   - Resource types found count
   - Total resources count
   - Errors encountered count with reference to log file

CSV output:

- Standard CSV format
- Header row with five columns
- Data rows sorted as specified
- One file per execution

Error log output:

- One error per line
- Timestamped entries
- Structured format for parsing if needed

## Testing Checklist

Bash script testing:

- Automatically installs Python 3.11 if not present
- Creates or updates .python-version file
- Activates Python 3.11 in current directory
- Verifies Python 3.11 is active before proceeding
- Installs packages without verbose output
- Reports total execution time
- Exits with proper codes on errors

Python script testing:

- Shows all prerequisite checks before stopping
- Exits cleanly if prerequisites fail
- Handles throttling without crashing
- Continues on permission errors
- Outputs correct CSV format and filename
- Error file contains all encountered errors
- Completes successfully with partial data on regional failures
- CSV filename includes alias when present, excludes when absent
- Execution completes within reasonable time for 15-20 regions

## Dependencies

External (must be pre-installed):

- pyenv

Managed by script:

- Python 3.11.x (automatically installed via pyenv if needed)
- boto3 (latest via pip)
- aioboto3 (latest via pip)

## Additional Notes

Design considerations:

- Script designed for local macOS execution only
- Requires active AWS credentials in standard locations
- Respects AWS API rate limits via retry configuration
- No cross-region aggregation (maintains region-level granularity)
- Output used by downstream applications for detailed resource enumeration
- Error handling prioritizes completing with partial data over failing completely
- Async implementation significantly reduces total execution time
- CSV format chosen for easy consumption by other tools
- Bash script handles environment setup autonomously
- Only requires human intervention if pyenv itself is not installed
