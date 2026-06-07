# Savanna / TigerGraph Workspace Automation Framework

[![Python Version](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/)
[![Test Framework](https://img.shields.io/badge/test__framework-Pytest-green.svg)](https://docs.pytest.org/)
[![Report Generation](https://img.shields.io/badge/reports-pytest--html-orange.svg)](https://github.com/pytest-dev/pytest-html)

## 📌 Overview

This repository houses a production-ready, **Python Pytest-based API Automation Framework** engineered to validate the dynamic lifecycle states of Savanna and TigerGraph workspaces. The primary objective is to rigorously test, verify, and audit **auto-suspend** and **auto-resume** behaviors, ensuring optimal resource utilization, system reliability, and accurate state transitions.

### Key Validation Domains
* **Lifecycle Automation:** Verification of idle thresholds, minimum/maximum auto-suspend timers, and keep-alive resets.
* **Elasticity & Recovery:** Evaluation of auto-resume triggers via API requests and sub-second operational readiness post-wakeup.
* **Query Integrity:** Validation of RESTPP query execution handling during transitional states (e.g., long-running queries blocking suspension).
* **Fault Tolerance:** Robust coverage of negative payloads, authorization failures, and boundary conditions.

---

## 🛠️ Tech Stack

| Component | Technology | Purpose |
| :--- | :--- | :--- |
| **Language** | Python 3.x | Core automation scripting |
| **Test Runner** | Pytest | Test suite orchestration, fixtures, and assertions |
| **HTTP Client** | Requests | Synchronous API interaction and payload delivery |
| **Config Management** | python-dotenv | Secure environment variable and credential handling |
| **Reporting Engine** | pytest-html | Formatted, self-contained HTML test report generation |

---

## 📂 Project Structure

```text
savanna_auto_suspend_tests/
│
├── tests/                          # Automated Test Suites
│   ├── test_debug_connection.py    # Health checks & API ping
│   ├── test_auto_suspend.py        # Suspend logic & idle timer validation
│   ├── test_auto_resume.py         # Resume triggers & post-wakeup verification
│   ├── test_negative_cases.py      # Edge cases, bounds, and error handling
│   └── test_state_transition.py    # End-to-end lifecycle state machine checks
│
├── utils/                          # Core Framework Utilities
│   ├── savanna_client.py           # Wrapped HTTP client for Savanna/TigerGraph APIs
│   └── wait_utils.py               # Explicit/Polling wait mechanisms for state changes
│
├── reports/                        # Test Execution Artifacts (Git ignored)
│   ├── test_report.html
│   ├── auto_suspend_report.html
│   └── negative_report.html
│
├── .env                            # Environment secrets (Base URL, API Keys)
├── conftest.py                     # Global Pytest fixtures and hooks
├── pytest.ini                      # Pytest CLI configurations and custom markers
├── requirements.txt                # Third-party dependencies
└── README.md                       # Project documentation

```

---

## 🚀 Getting Started

### 1. Prerequisites

Ensure you have Python installed locally.

### 2. Installation

Clone the repository and install the required dependencies:

```bash
pip install -r requirements.txt

```

### 3. Environment Configuration

Create a `.env` file in the root directory based on your environment specifications:

```env
SAVANNA_BASE_URL=[https://api.savanna.tigergraph.com](https://api.savanna.tigergraph.com)
API_KEY=your_secure_api_key_here
WORKSPACE_ID=your_target_workspace_id

```

---

## 🧪 Test Execution Matrix

Tests can be executed modularly or as a comprehensive regression suite. The `-v` (verbose) and `-s` (allow stdout) flags are recommended for real-time log tracking.

### Target Execution Commands

| Target Scope | Execution Command | Description |
| --- | --- | --- |
| **Sanity / Health Check** | `pytest tests/test_debug_connection.py -v -s` | Verifies API reachability and authentication. |
| **Auto-Suspend Suite** | `pytest tests/test_auto_suspend.py -v -s` | Validates idle time configurations and timers. |
| **Auto-Resume Suite** | `pytest tests/test_auto_resume.py -v -s` | Validates waking behaviors from a suspended state. |
| **Negative Testing** | `pytest tests/test_negative_cases.py -v -s` | Asserts proper HTTP error status codes. |
| **State Transitions** | `pytest tests/test_state_transition.py -v -s` | Evaluates multi-state sequencing (Active ↔ Suspended). |
| **Full Regression** | `pytest -v -s` | Executes the entire test catalog. |

### 📊 Report Generation

To execute the full suite and generate a centralized, shareable **HTML report** containing execution logs and metadata, run:

```bash
pytest -v -s --html=reports/test_final_report.html --self-contained-html

```

---

## 📋 Test Case Specifications

### 🔹 Auto-Suspend Scenarios

| TC ID | Scenario Description | Expected Result |
| --- | --- | --- |
| **TC01** | Verify workspace API infrastructure reachability | Returns `HTTP 200 OK` with valid response metadata. |
| **TC02** | Verify minimum acceptable auto-suspend configuration | Lower boundary value accepted and configured successfully. |
| **TC03** | Verify workspace does not suspend prior to configured window | Workspace state remains `ACTIVE` throughout the threshold countdown. |
| **TC04** | Verify keep-alive activity resets the internal suspend timer | Active traffic resets countdown; prevents premature suspension. |
| **TC05** | Verify long-running RESTPP queries prevent auto-suspension | Suspension blocked until active query executions terminate. |
| **TC06** | Verify manual trigger of workspace suspension | Immediate transition to `SUSPENDING`/`SUSPENDED` state via API. |

### 🔹 Auto-Resume Scenarios

| TC ID | Scenario Description | Expected Result |
| --- | --- | --- |
| **TC07** | Verify workspace auto-resumes gracefully from an API request | State moves to `RESUMING`, then stabilizes to `ACTIVE`. |
| **TC08** | Verify query execution capability immediately after resume | RESTPP query executes successfully post-wakeup. |
| **TC09** | Verify workspace remains stable and active post-resume | No immediate regression or cyclical re-suspension observed. |
| **TC10** | Verify edge-case system behavior during resume timeouts | Framework gracefully captures and reports network/gateway timeouts. |

### 🔹 Negative & Boundary Scenarios

| TC ID | Scenario Description | Expected Result |
| --- | --- | --- |
| **TC11** | Execute request using an invalid or expired API Key | Returns `HTTP 401 Unauthorized`. |
| **TC12** | Execute target request against a non-existent Workspace ID | Returns `HTTP 404 Not Found`. |
| **TC13** | Attempt configuring auto-suspend time *below* allowed minimum | Returns `HTTP 400 Bad Request` with semantic validation error. |
| **TC14** | Attempt configuring auto-suspend time *above* allowed maximum | Returns `HTTP 400 Bad Request` with semantic validation error. |
| **TC15** | Submit corrupted/malformed RESTPP query payloads or endpoints | System rejects payload safely with error tracking context. |

### 🔹 State Transition Scenarios

| TC ID | Scenario Description | Expected Result |
| --- | --- | --- |
| **TC16** | Validate sequential `ACTIVE` ➔ `SUSPENDED` transition phases | Accurate logging of intermediated state flags. |
| **TC17** | Validate sequential `SUSPENDED` ➔ `ACTIVE` transition phases | State engine records consistent metrics across lifecycle. |
| **TC18** | Evaluate concurrent/multiple rapid resume API request sequences | System handles debouncing gracefully; avoids race conditions. |
| **TC19** | Assert overall workspace status data consistency across nodes | State values match across global system parameters. |
| **TC20** | Verify debug connection health endpoint | Returns `HTTP 200 OK` with valid service health metadata.Not Counted as actual test cases |



```

```