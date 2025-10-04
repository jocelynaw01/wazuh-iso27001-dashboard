## 1.0 Introduction
### 1.1 Purpose of the Project
The purpose of the ISO 27001:2022 Wazuh Dashboard is to provide a structured and centralized view of security events mapped to Annex A controls. This project enables organizations to:
- Provide a centralized view of security events mapped to ISO 27001:2022 Annex A controls.
- Enable real-time monitoring of compliance-related activities, such as authentication failures, privilege changes, and audit policy modifications.
- Support evidence collection for internal and external ISO 27001 audits.
- Help security teams quickly identify gaps in control coverage by mapping rules to specific ISO controls.
- Facilitate continuous compliance monitoring instead of point-in-time assessments.
- Provide metrics and trends (per control, per asset, per severity) that can be reported to auditors and management.

### 1.2 Overview of Annex A Controls
Annex A in ISO 27001:2022 defines a reference set of security controls required by Clause 6.1.3 (Information security risk treatment) and documented in the organization’s Statement of Applicability (SoA). Annex A consists of 93 controls grouped into four domains. 
Table 1 summarizes the domains, number of controls, ranges, and focus areas.
| Domains       | Number of Controls | Control Ranges | Focus Areas  |
|:--------------|:-------------------|:---------------|:-------------|
|Organizational |         37         | A.5.1 – A.5.37 | Policies, processes, organizational structures, supplier management, governance, roles, responsibilities |
|People         |          8         | A.6.1 – A.6.8  | Human resources security, personnel management, awareness and training, responsibilities during and after employment. |
|Physical       |         14         | A.7.1 – A.7.14 | Facility entry systems, asset protection, secure disposal, storage media handling, clear desk/clear screen, environmental protection. |
|Technological  |         34         | A.8.1 – A.8.34 | Authentication, encryption, logging, monitoring, backups, vulnerability management, incident response, technical configuration. |

*Table 1: Overview of ISO 27001:2022 Annex A Controls*

> Note: Not all Annex A controls can be implemented in Wazuh. For example, physical security measures such as A.7.3 (Physical entry controls) are outside the scope of Wazuh monitoring. These controls are excluded from this dashboard design.

## 2.0 Prerequisites
- Installed Wazuh Manager and Indexer (Elasticsearch/OpenSearch).
- Installed Wazuh Dashboard (Kibana/OpenSearch Dashboards).
- Familiarity with ISO 27001:2022 Annex A controls.

## 3.0 Tagging Rules for ISO 27001:2022
Wazuh rules can be extended with ISO/IEC 27001:2022 compliance tags. These tags enable filtering, searching, and visualizing events by specific Annex A controls in the Wazuh Dashboard.

### 3.1 Mapping Rules to Controls
Each Wazuh rule may correspond to one or more Annex A controls. For example, **Rule ID 5710** (SSH login attempt with a non-existent user) relates to:
- A.5.15 Access Control – Ensure that access to information and systems is restricted to authorized users only.
- A.5.16 Identity Management – Manage identities and their associated access rights consistently throughout the user lifecycle.
- A.8.5 Secure Authentication – Enforce secure authentication methods to verify user identities.
- A.8.15 Logging – Record authentication attempts to support monitoring, investigation, and compliance reporting.

### 3.2 Tagging Syntax
Compliance tags are added inside the <group> element of the rule definition. Use the following format for consistency:
```
iso_27001_2022_<control_number>
```

For example:
```
<rule id="5710" level="5">
    <if_sid>5700</if_sid>
    <match>illegal user|invalid user</match>
    <description>sshd: Attempt to login using a non-existent user</description>
    <mitre>
      <id>T1110.001</id>
      <id>T1021.004</id>
    </mitre>
    <group>authentication_failed,iso_27001_2022_5.15,iso_27001_2022_5.16,iso_27001_2022_8.5,iso_27001_2022_8.15,</group>
</rule>
```

### 3.3 Guidelines for Tagging
- Use consistent tag naming (iso_27001_2022_X.X) across all rules.
- A single rule can include multiple Annex A controls if applicable.
- Apply tags at the most granular level (e.g., 8.15 instead of just 8).
- Document mappings in an external Excel/CSV file for maintenance and audit traceability.

### 3.4 Verification
After updating rules:
1. Restart Wazuh Manager:
```
systemctl restart wazuh-manager
```
2. Verify the mapping by filtering in the Wazuh Dashboard. For example:
```
rule.id: 5710 AND rule.groups: iso_27001_2022_5.15
```
If the rule appears in the search results, the tagging was applied successfully.

## 4.0 Dashboard Design
Wazuh provides multiple visualization options to build dashboards that align events with ISO/IEC 27001:2022 controls. This section explains how to design dashboards for both an overall compliance overview and specific Annex A control categories.
For additional guidance on building visualizations, refer to the Wazuh official documentation: [Creating custom dashboards](https://documentation.wazuh.com/current/user-manual/wazuh-dashboard/creating-custom-dashboards.html).

### 4.1 Dashboard for Overall ISO 27001:2022 Coverage
Steps:
1. Navigate to **Explore → Dashboards**.
2. Click **+ Create Dashboard**, then **+ Create new**.
3. In the **New Visualization** modal, select **Pie**, and choose the index pattern `wazuh-alerts-*`.
4. Under **Metrics**, set **Aggregation** to **Count**.
5. Click on **+Add** under **Buckets** and select `Split slices`, and use the following settings:
- Aggregation is `Terms`.
- Field is `rule.groups`.
- Size is `10` (or adjust as needed).
6. In the Advanced section, set a filter `iso_27001_2022.*`. This ensures only rules with ISO 27001:2022 tags are included.
7. Click **Update**.
8. **Save** the visualization with a clear title (e.g., ISO 27001:2022 Control Distribution).
9. Repeat as needed to add complementary visualizations (e.g., bar charts by agent, time series by severity).
10. **Save** the dashboard with a descriptive name (e.g., ISO/IEC 27001 Dashboard).

### 4.2 Dashboards by Control Category
To improve usability, Annex A controls can be grouped into thematic dashboards. Table 2 shows an example grouping used in this project.
| Dashboard Themes       | Annex A Controls |
|:--------------|:-------------------|
| Access Control & Identity Management | A.5.3 Segregation of Duties |
|  | A.5.10 Acceptable Use of Information and Other Associated Assets |
|  | A.5.15 Access Control |
|  | A.5.18	Access Rights |
| Cloud Control	 | A.5.23	Information Security for Use of Cloud Services |
| Data Handling & Protection | A.5.14 Information Transfer |
|  | A.8.10	Information Deletion |
|  | A.8.12	Data Leakage Prevention |
| Endpoint & Device Security | A.6.7 Remote Working |
|  | A.8.1 User Endpoint Devices |
|  | A.8.9 Configuration Management |
|  | A.8.19 Installation of Software on Operational Systems |
| Incident Control | A.5.26	Response to Information Security Incidents |
| Monitor & Logging | A.5.22 Monitoring, Review and Change Management of Supplier Services |
|  | A.8.15	Logging |
|  | A.8.16	Monitoring Activities |
|  | A.8.16	Monitoring Activities |
|  | A.8.32	Change Management |
|  | A.8.34	Protection of Information Systems during Audit Testing |
| Threat Intelligence | A.5.7 Threat Intelligence |
| Vulnerability & Technical Security Management | A.8.7	Protection Against Malware |
|  | A.8.8 Management of Technical Vulnerabilities |
|  | A.8.20	Networks Security |
|  | A.8.21	Security of Network Services |
|  | A.8.23	Web Filtering |
|  | A.8.26	Application Security Requirements |

Steps:
1.	Navigate to **Explore** → **Dashboards**.
2.	Click **+ Create Dashboard**, then **+ Create new**.
3.	In the **New Visualization** modal, select **Pie**, and choose the index pattern `wazuh-alerts-*`.
4.	Under **Metrics**, set **Aggregation** to **Count**.
5.	Click on **+Add** under **Buckets** and select `Split slices`, and use the following settings:
- Aggregation is `Terms`.
- Field is `agent.name`.
- Size is `10` (or adjust as needed).
6.	Click **Update**.
7.	Click **+ Add Filter**, and use the following settings:
- Field is `rule.groups`.
- Operator is `is one of`.
- Values: List the ISO/IEC 27001 control tags for the category (e.g., `iso_27001_2022_8.15`, `iso_27001_2022_8.16`).
8. Click **Save**.
9. **Save** the visualization with a clear title (e.g., Access Control Events by Agent).
10.	Continue adding visualizations (e.g., trends, top rules, severity distribution).
11. Save the completed dashboard under the relevant category name (e.g. Access Control & Identity Management).

## Appendix
The detailed mapping between ISO 27001:2022 Annex A controls and Wazuh rules is maintained in a separate Excel file. This file enables filtering, sorting, and updating as the ruleset evolves. You can download it here: [ISO 27001:2022 - Wazuh Rules Mapping (Community)](https://docs.google.com/spreadsheets/d/1jtU_i-qM5rAHELA0m8ghDZiqDkbKiBE5/edit?usp=sharing&ouid=104104054315219906721&rtpof=true&sd=true).

### Usage Instructions
- **Filtering by Control**: Use Excel filters to display all rules mapped to a specific Annex A control.
- **Filtering** by Rule: Select a rule ID to view the control(s) it supports.
- **Audit Preparation**: Export the filtered dataset as CSV/PDF to provide evidence of compliance mappings.
- **Maintenance**: Update the file when new Wazuh rules are introduced or when control mappings are revised.

