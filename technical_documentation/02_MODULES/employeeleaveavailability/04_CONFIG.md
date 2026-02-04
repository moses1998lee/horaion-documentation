# Leave & Availability Configuration

## Database Configuration

The **Leave Module** manages the `employee_leave_availability` table.

### Indexes & Performance
Performance is critical here because the Scheduler queries this table *thousands* of times during a generation run (checking every employee for every day).

| Index Name | Column | Purpose |
| :--- | :--- | :--- |
| `idx_leave_availability_employee_id` | `employee_id` | **Primary Lookup**: Fetching constraints for a specifc person. |
| `idx_leave_availability_dates` | `start_date`, `end_date` | **Range Queries**: "Who is on leave next week?" |
| `idx_leave_availability_status` | `status` | Filtering `APPROVED` vs `PENDING`. |

## Application Properties

No specific external API keys are required.

### Event Listeners
Verify that your `LeaveEmailTemplateService` is configured if you expect email notifications.
*   **Template Path**: Ensure the HTML templates for "Leave Approved" exist in your resources folder.
