# JARVIS OS — Hệ Điều Hành Doanh Nghiệp AI

**Phiên bản tài liệu:** 1.0  
**Loại tài liệu:** Tài liệu đặc tả kiến trúc khái niệm và định hướng triển khai  
**Mục đích:** Giúp người đọc hiểu rõ ý tưởng, hình dung kiến trúc tổng thể, phạm vi tính năng phiên bản 1.0, các bài toán có thể giải quyết, hướng triển khai kỹ thuật và các thách thức cần xử lý khi xây dựng hệ thống.

---

## Mục lục

1. [Tổng quan ý tưởng](#1-tổng-quan-ý-tưởng)
2. [Bối cảnh và vấn đề cần giải quyết](#2-bối-cảnh-và-vấn-đề-cần-giải-quyết)
3. [Tầm nhìn hệ thống](#3-tầm-nhìn-hệ-thống)
4. [Nguyên tắc thiết kế cốt lõi](#4-nguyên-tắc-thiết-kế-cốt-lõi)
5. [Định vị sản phẩm](#5-định-vị-sản-phẩm)
6. [Kiến trúc tổng thể](#6-kiến-trúc-tổng-thể)
7. [Các khái niệm nền tảng](#7-các-khái-niệm-nền-tảng)
8. [Các phân hệ chính](#8-các-phân-hệ-chính)
9. [Tính năng dự kiến trong phiên bản 1.0](#9-tính-năng-dự-kiến-trong-phiên-bản-10)
10. [Các bài toán có thể giải quyết nếu triển khai thành công](#10-các-bài-toán-có-thể-giải-quyết-nếu-triển-khai-thành-công)
11. [Luồng hoạt động mẫu](#11-luồng-hoạt-động-mẫu)
12. [Hướng triển khai kỹ thuật](#12-hướng-triển-khai-kỹ-thuật)
13. [Đề xuất kiến trúc dữ liệu](#13-đề-xuất-kiến-trúc-dữ-liệu)
14. [Đề xuất kiến trúc bảo mật và kiểm soát](#14-đề-xuất-kiến-trúc-bảo-mật-và-kiểm-soát)
15. [AgentOps, giám sát và kiểm toán](#15-agentops-giám-sát-và-kiểm-toán)
16. [Những khó khăn kỹ thuật cần giải quyết](#16-những-khó-khăn-kỹ-thuật-cần-giải-quyết)
17. [Lộ trình triển khai đề xuất](#17-lộ-trình-triển-khai-đề-xuất)
18. [Kết luận](#18-kết-luận)

---

# 1. Tổng quan ý tưởng

**Jarvis OS** là một nền tảng vận hành doanh nghiệp bằng AI, trong đó doanh nghiệp có thể tạo lập, quản trị, điều phối và giám sát các **AI Agent** như một đội ngũ nhân sự số.

Khác với một chatbot hoặc một AI Agent đơn lẻ, Jarvis OS hướng tới mô hình **nền tảng quản trị và điều phối đa tác nhân**. Người dùng có thể mô tả nhu cầu bằng ngôn ngữ tự nhiên, sau đó hệ thống sẽ phân tích, đề xuất agent, đề xuất công cụ, lập kế hoạch thực thi, đánh giá rủi ro, trình người có thẩm quyền phê duyệt và đưa agent vào vận hành trong phạm vi được kiểm soát.

Ví dụ, người dùng chỉ cần nói:

```text
Hàng ngày lúc 8h sáng, lấy báo cáo tỷ giá USD/VND, EUR/VND, JPY/VND và gửi vào Telegram cho tôi.
```

Jarvis OS có thể tự động:

1. Phân tích đây là nhu cầu báo cáo định kỳ.
2. Chọn hoặc tạo một agent chuyên trách.
3. Kiểm tra có sẵn công cụ lấy tỷ giá và gửi Telegram hay chưa.
4. Nếu thiếu công cụ, đề xuất tạo công cụ mới.
5. Tạo bản kế hoạch thực thi.
6. Trình người quản lý phê duyệt.
7. Chạy thử trong sandbox.
8. Bật lịch chạy định kỳ.
9. Gửi báo cáo theo lịch.
10. Ghi log, audit và chi phí vận hành.

Điểm cốt lõi không phải là để AI tự do làm mọi thứ, mà là tạo ra một cơ chế để **AI tự kiến tạo năng lực vận hành mới cho doanh nghiệp**, trong khi con người, chính sách, sandbox và cơ chế kiểm toán vẫn giữ quyền kiểm soát cuối cùng.

---

# 2. Bối cảnh và vấn đề cần giải quyết

## 2.1. Doanh nghiệp có nhiều tác vụ lặp lại nhưng khó tự động hóa

Trong thực tế, nhiều công việc của doanh nghiệp mang tính lặp lại:

- tổng hợp báo cáo hằng ngày;
- kiểm tra trạng thái hệ thống;
- đọc email và phân loại yêu cầu;
- lấy dữ liệu từ nhiều nguồn;
- tạo file import;
- kiểm tra dữ liệu thiếu, trùng, sai định dạng;
- giám sát lịch hết hạn chứng chỉ SSL;
- tổng hợp log lỗi;
- soạn báo cáo tiến độ;
- kiểm tra tuân thủ tài liệu, quy trình, biểu mẫu.

Các tác vụ này thường không quá phức tạp đối với con người, nhưng lại khó tự động hóa hoàn toàn vì cần hiểu ngữ cảnh, cần kết nối nhiều hệ thống, cần xử lý ngoại lệ và cần kiểm soát rủi ro.

## 2.2. Không phải người dùng nào cũng biết code hoặc biết cấu hình hệ thống

Một lãnh đạo, quản lý nghiệp vụ hoặc chuyên viên vận hành có thể biết rõ mình cần gì, nhưng không biết:

- viết script;
- gọi API;
- cấu hình cron job;
- tạo Telegram Bot;
- viết pipeline xử lý dữ liệu;
- kết nối database;
- định nghĩa workflow;
- triển khai job chạy nền.

Jarvis OS giải quyết khoảng cách này bằng cách cho phép người dùng mô tả nhu cầu tự nhiên, còn hệ thống tự tạo năng lực cần thiết theo cách có kiểm soát.

## 2.3. AI Agent mạnh nhưng khó đưa vào môi trường doanh nghiệp

AI Agent có thể đọc, phân tích, lập kế hoạch, gọi công cụ và tạo kết quả. Tuy nhiên, khi đưa vào môi trường doanh nghiệp, các câu hỏi lớn xuất hiện:

- Agent được phép làm gì?
- Agent có được đọc dữ liệu nội bộ không?
- Agent có được ghi dữ liệu vào hệ thống chính không?
- Nếu agent sai thì phát hiện thế nào?
- Ai phê duyệt?
- Có rollback không?
- Có audit không?
- Có kiểm soát chi phí token không?
- Có ngăn được vòng lặp vô hạn không?
- Có cách nào biết agent đã gọi tool nào, dùng input gì, trả output gì không?

Jarvis OS được thiết kế để giải quyết các vấn đề này bằng một nền tảng quản trị agent có kỷ luật.

---

# 3. Tầm nhìn hệ thống

Jarvis OS là một **Enterprise AgentOps Governance Platform** — nền tảng vận hành, quản trị và kiểm soát đội ngũ AI Agent trong doanh nghiệp.

Tầm nhìn của hệ thống:

```text
Người dùng mô tả nhu cầu bằng ngôn ngữ tự nhiên.
AI đề xuất năng lực mới cần tạo ra.
Hệ thống sinh agent, tool, workflow và chính sách kiểm soát.
Con người phê duyệt.
Agent vận hành trong sandbox, policy, quota và audit.
Doanh nghiệp tích lũy dần một đội ngũ nhân sự số có kiểm soát.
```

Triết lý cốt lõi:

> **AI được quyền sáng tạo năng lực mới, nhưng quyền thực thi và quyền cam kết dữ liệu chính thức phải được kiểm soát bởi con người, policy, sandbox và audit.**

---

# 4. Nguyên tắc thiết kế cốt lõi

## 4.1. AI đề xuất, con người phê duyệt

Agent có thể phân tích yêu cầu, lập kế hoạch, đề xuất tool, đề xuất quyền hạn và đề xuất lịch chạy. Tuy nhiên, agent không được tự cấp quyền cho chính mình.

Luồng chuẩn:

```text
User Request
→ AI phân tích nhu cầu
→ AI đề xuất agent/tool/workflow
→ Hệ thống đánh giá rủi ro
→ AI lập tờ trình kế hoạch
→ CEO/Manager phê duyệt
→ Runtime triển khai trong phạm vi được duyệt
→ AgentOps ghi audit và giám sát
```

## 4.2. Không chỉ kiểm soát agent được làm gì, mà kiểm soát agent được làm đến mức nào

Một agent có thể thực hiện hành động ở nhiều mức khác nhau:

- chỉ đọc dữ liệu;
- chỉ tạo bản nháp;
- tạo đề xuất thay đổi;
- tạo file import;
- ghi vào staging;
- ghi chính thức sau khi được duyệt;
- ghi trực tiếp vào hệ thống chính.

Doanh nghiệp có thể chọn mức phù hợp với rủi ro của từng bài toán.

## 4.3. Tách generate và commit

Nguyên tắc quan trọng:

```text
Agent generates.
Human verifies.
System commits.
```

Hoặc diễn đạt theo ngữ cảnh doanh nghiệp:

> **AI tạo năng suất. Con người giữ quyền cam kết.**

Ví dụ, thay vì để agent cập nhật trực tiếp báo cáo công việc vào dữ liệu doanh nghiệp, agent có thể tạo file import hoặc ghi vào bảng staging. Người quản lý kiểm tra và thực hiện import hoặc duyệt promote dữ liệu sang vùng chính thức.

## 4.4. Mọi công cụ đều phải có contract

Tool không nên chỉ là một đoạn code. Mỗi tool cần có:

- tên;
- mô tả;
- input schema;
- output schema;
- quyền truy cập network;
- quyền truy cập file;
- quyền truy cập database;
- quyền đọc secret;
- timeout;
- quota;
- retry policy;
- log policy.

## 4.5. Mọi hành động quan trọng đều phải có audit

Hệ thống cần ghi lại:

- ai yêu cầu;
- agent nào xử lý;
- kế hoạch nào được sinh ra;
- ai phê duyệt;
- tool nào được gọi;
- tham số gọi tool;
- kết quả tool;
- dữ liệu đầu ra;
- chi phí;
- lỗi;
- thời gian chạy;
- trạng thái cuối cùng.

Không nên phụ thuộc vào việc lưu “luồng suy nghĩ nội bộ” của model. Thay vào đó cần lưu **Decision Trace** và **Execution Trace**.

---

# 5. Định vị sản phẩm

Jarvis OS không nên được định vị là:

```text
Một chatbot thông minh hơn
```

Cũng không nên chỉ được định vị là:

```text
Một framework tạo nhiều AI Agent
```

Định vị phù hợp hơn:

```text
Jarvis OS là nền tảng vận hành đội ngũ AI Agent trong doanh nghiệp,
cho phép AI tự tạo năng lực mới,
nhưng mọi hành động, dữ liệu, tool, memory và quyền thực thi
đều được kiểm soát bằng policy, approval, sandbox và audit.
```

Điểm khác biệt chính:

1. Tạo agent bằng ngôn ngữ tự nhiên.
2. Tạo hoặc đề xuất tool bằng AI.
3. Có cơ chế phê duyệt trước khi thực thi.
4. Có Action Mode để giới hạn mức độ hành động.
5. Có vùng đệm hành động: draft, import file, staging, pending approval.
6. Có memory phân cấp và quản trị tri thức.
7. Có AgentOps để giám sát, audit, replay và kiểm soát chi phí.

---

# 6. Kiến trúc tổng thể

## 6.1. Sơ đồ phân lớp

```text
┌────────────────────────────────────────────────────────────┐
│ 1. Interaction Layer                                       │
│    GUI / CLI / Telegram / Webhook / API / Daemon           │
├────────────────────────────────────────────────────────────┤
│ 2. CEO Gate & Approval Layer                               │
│    Plan Approval / Human Review / Step-by-step Control     │
├────────────────────────────────────────────────────────────┤
│ 3. Agent Governance Layer                                  │
│    Policy / Permission / Budget / Risk Classification      │
├────────────────────────────────────────────────────────────┤
│ 4. HR & Orchestration Layer                                │
│    Agent Registry / Workflow Graph / Squad Coordination    │
├────────────────────────────────────────────────────────────┤
│ 5. Agent Runtime Layer                                     │
│    Agent Instance / Persona / Model Adapter / State        │
├────────────────────────────────────────────────────────────┤
│ 6. Tool Runtime & MCP Layer                                │
│    Tool Registry / MCP Client / Sandbox / Tool Forge       │
├────────────────────────────────────────────────────────────┤
│ 7. Memory & Knowledge Layer                                │
│    Instance Memory / Squad Memory / Global Knowledge Base  │
├────────────────────────────────────────────────────────────┤
│ 8. AgentOps & Audit Layer                                  │
│    Trace / Metrics / Cost / Replay / Evaluation / Rollback │
└────────────────────────────────────────────────────────────┘
```

## 6.2. Diễn giải các lớp

| Lớp | Vai trò |
|---|---|
| Interaction Layer | Cung cấp kênh tương tác cho người dùng và hệ thống ngoài |
| CEO Gate & Approval Layer | Quản lý phê duyệt, kiểm tra kế hoạch, can thiệp thủ công |
| Agent Governance Layer | Quản lý chính sách, quyền, ngân sách, mức rủi ro |
| HR & Orchestration Layer | Quản lý vòng đời agent và điều phối workflow |
| Agent Runtime Layer | Chạy agent, quản lý model, persona, state |
| Tool Runtime & MCP Layer | Quản lý tool, sandbox, MCP connector, dynamic tool generation |
| Memory & Knowledge Layer | Quản lý bộ nhớ cá nhân, nhóm và tri thức toàn cục |
| AgentOps & Audit Layer | Giám sát, trace, replay, đánh giá, chi phí và kiểm toán |

---

# 7. Các khái niệm nền tảng

## 7.1. Agent

Agent là một thực thể AI có vai trò, mục tiêu, quyền hạn, công cụ, bộ nhớ và chính sách vận hành riêng.

Ví dụ:

```yaml
agent:
  id: daily_exchange_rate_reporter
  name: Daily Exchange Rate Reporter
  role: Chuyên viên báo cáo tỷ giá hằng ngày
  owner: finance_manager
  model: gpt-4.1-mini
  action_mode: send_notification
  status: active
```

## 7.2. Agent Blueprint

Blueprint là mẫu agent được chuẩn hóa. Agent cụ thể được sinh ra từ blueprint.

Ví dụ blueprint:

- Scheduled Report Agent;
- Monitoring Agent;
- Data Quality Checking Agent;
- Document Processing Agent;
- DevOps Assistant Agent;
- Customer Support Classification Agent;
- Compliance Review Agent.

Blueprint giúp hệ thống không sinh agent một cách tùy tiện, mà sinh từ các mẫu đã được kiểm soát.

## 7.3. Agent Instance

Agent Instance là một agent cụ thể được tạo ra từ blueprint hoặc prototype.

Ví dụ:

```text
Prototype: Scheduled Report Agent

Instance 1: Daily Exchange Rate Reporter
Instance 2: Daily Sales Summary Agent
Instance 3: Daily Infrastructure Health Reporter
```

Mỗi instance có model, persona, tool, memory scope, quyền hạn, lịch chạy và owner riêng.

## 7.4. Tool

Tool là năng lực bên ngoài mà agent có thể sử dụng, ví dụ:

- gọi API;
- gửi Telegram;
- đọc file;
- truy vấn database;
- tạo file Excel;
- kiểm tra trạng thái server;
- gọi webhook;
- tạo ticket;
- gửi email.

Tool phải có contract và được đăng ký trong Tool Registry.

## 7.5. Tool Contract

Tool Contract định nghĩa rõ tool làm gì, nhận input gì, trả output gì và được phép truy cập gì.

Ví dụ:

```yaml
tool:
  name: exchange_rate_fetcher
  description: Lấy tỷ giá từ API công khai đã được phê duyệt

input_schema:
  type: object
  properties:
    base_currency:
      type: string
    target_currencies:
      type: array
      items:
        type: string

output_schema:
  type: object
  properties:
    rates:
      type: object
    timestamp:
      type: string

permissions:
  network:
    allow_domains:
      - api.exchangerate.host
  filesystem:
    read: []
    write: []
  database:
    access: none
  secrets:
    read: []

runtime:
  timeout_seconds: 30
  memory_limit_mb: 128
  max_retries: 2
```

## 7.6. Action Mode

Action Mode xác định agent được hành động đến mức nào.

| Action Mode | Ý nghĩa | Mức rủi ro |
|---|---|---:|
| `read_only` | Chỉ đọc và phân tích | Thấp |
| `draft_only` | Chỉ tạo bản nháp | Thấp |
| `proposal_only` | Chỉ tạo đề xuất thay đổi | Thấp |
| `generate_import_file` | Sinh file import để người dùng xử lý | Thấp - Trung bình |
| `write_to_staging` | Ghi vào vùng staging/draft | Trung bình |
| `commit_with_approval` | Ghi chính thức sau khi được duyệt | Trung bình - Cao |
| `direct_commit` | Tự ghi chính thức vào dữ liệu lõi | Cao |
| `direct_execute` | Tự thực thi hành động hệ thống | Rất cao |

## 7.7. Action Buffer

Action Buffer là vùng đệm hành động giữa kết quả agent tạo ra và hệ thống chính thức.

Các dạng buffer:

- draft document;
- import file;
- staging table;
- pending approval queue;
- change request;
- dry-run result;
- simulation report;
- patch proposal.

## 7.8. CEO Approval Package

CEO Approval Package là tờ trình do hệ thống sinh ra để người có thẩm quyền phê duyệt trước khi triển khai agent, tool hoặc workflow.

Nội dung gồm:

- mục tiêu;
- agent cần tạo;
- tool cần dùng;
- quyền cần cấp;
- Action Mode;
- lịch chạy;
- chi phí dự kiến;
- rủi ro;
- biện pháp kiểm soát;
- kết quả chạy thử nếu có.

## 7.9. Memory

Memory là bộ nhớ của hệ thống, chia thành:

- Instance Memory: bộ nhớ riêng của agent;
- Squad Memory: bộ nhớ dùng chung trong nhóm agent;
- Global Knowledge Base: tri thức toàn cục, quy định, tiêu chuẩn, tài liệu bắt buộc tuân thủ.

Memory cần có metadata quản trị, không chỉ là vector embedding.

---

# 8. Các phân hệ chính

## 8.1. Interaction Layer

Phân hệ này cung cấp các kênh để người dùng và hệ thống ngoài tương tác với Jarvis OS.

Thành phần chính:

- GUI Dashboard;
- CLI Executive;
- REST API;
- Telegram Bot Connector;
- Webhook Receiver;
- Background Daemon.

Tính năng:

- tạo yêu cầu mới;
- xem danh sách agent;
- xem trạng thái agent;
- duyệt kế hoạch;
- pause/resume/kill agent;
- xem log và kết quả chạy;
- cấu hình kết nối ngoài.

## 8.2. CEO Gate & Approval Layer

Phân hệ này đảm bảo các hành động quan trọng phải được người có thẩm quyền phê duyệt.

Tính năng:

- Draft Plan Authorization;
- Approval Queue;
- Step-by-step Interactive Mode;
- Human Review;
- Approval History;
- Reject/Revise Request;
- Emergency Stop.

## 8.3. Agent Governance Layer

Phân hệ này quản lý chính sách và kiểm soát rủi ro.

Tính năng:

- Permission Policy;
- Budget Policy;
- Risk Classification;
- Tool Access Control;
- Data Access Scope;
- Action Mode Enforcement;
- Guardrail Policy;
- Circuit Breaker.

## 8.4. HR & Orchestration Layer

Phân hệ này quản lý vòng đời agent và phối hợp nhiều agent.

Tính năng:

- Agent Registry;
- Agent Blueprint Registry;
- Runtime Hot-Deployment;
- Undeploy & Deprovisioning;
- Workflow Graph;
- Sequential Workflow;
- Hierarchical Workflow;
- Network/Mesh Workflow;
- Scheduler.

## 8.5. Agent Runtime Layer

Phân hệ này thực thi agent.

Tính năng:

- Model Adapter;
- Persona Matrix;
- Prompt Contract;
- Agent State;
- Tool Binding;
- Memory Binding;
- Retry Policy;
- Timeout Control;
- Output Schema Enforcement.

## 8.6. Tool Runtime & MCP Layer

Phân hệ này quản lý công cụ và kết nối ngoại vi.

Tính năng:

- Tool Registry;
- MCP Client;
- Tool Contract;
- Dynamic Tool Forge;
- Sandbox Execution;
- Unit Test Runner;
- Static Scan;
- Secret Binding;
- Tool Versioning.

## 8.7. Memory & Knowledge Layer

Phân hệ này quản lý tri thức và bộ nhớ.

Tính năng:

- Instance Memory;
- Squad Memory;
- Global Knowledge Base;
- Vector Search;
- Metadata Search;
- Memory Approval;
- Memory TTL;
- Memory Conflict Detection;
- Memory Versioning.

## 8.8. AgentOps & Audit Layer

Phân hệ này giám sát và kiểm toán toàn bộ hệ thống.

Tính năng:

- Execution Trace;
- Decision Trace;
- Tool Call Trace;
- Cost Tracking;
- Latency Tracking;
- Failure Analysis;
- Replay;
- Evaluation;
- Audit Report;
- Policy Violation Alert.

---

# 9. Tính năng dự kiến trong phiên bản 1.0

Phiên bản 1.0 không nên ôm toàn bộ tham vọng của Jarvis OS. Mục tiêu của 1.0 là chứng minh được 4 năng lực chính:

1. Người dùng mô tả nhu cầu tự nhiên.
2. Hệ thống tạo agent từ blueprint.
3. Hệ thống chọn hoặc đề xuất tool.
4. Tất cả phải có approval, sandbox, audit và Action Mode.

## 9.1. Nhóm tính năng bắt buộc

### 9.1.1. Agent Registry

Quản lý danh sách agent:

- tạo agent;
- xem agent;
- cập nhật agent;
- pause/resume agent;
- archive agent;
- xem lịch sử chạy.

### 9.1.2. Agent Blueprint

Hỗ trợ tối thiểu các blueprint:

| Blueprint | Mục đích |
|---|---|
| Scheduled Report Agent | Tạo báo cáo định kỳ |
| Monitoring Agent | Kiểm tra trạng thái hệ thống |
| Data Import Assistant | Tạo file import hoặc staging data |
| Document Summary Agent | Tóm tắt tài liệu |

### 9.1.3. Tool Registry

Quản lý tool có sẵn:

- HTTP API Caller;
- Telegram Sender;
- Email Sender;
- File Writer;
- Excel/CSV Generator;
- Database Read-only Query Tool;
- Webhook Caller.

### 9.1.4. Tool Contract

Mỗi tool phải có:

- input schema;
- output schema;
- permission manifest;
- timeout;
- retry;
- log policy;
- secret requirement.

### 9.1.5. Approval Package

Hệ thống phải sinh tờ trình trước khi:

- tạo agent mới;
- bind tool mới;
- bật lịch chạy;
- cấp quyền đọc dữ liệu;
- tạo file import;
- ghi staging;
- gọi notification tool.

### 9.1.6. Action Mode

Phiên bản 1.0 nên hỗ trợ:

- `read_only`;
- `draft_only`;
- `proposal_only`;
- `generate_import_file`;
- `write_to_staging`;
- `send_notification`.

Không nên hỗ trợ mặc định:

- `direct_commit`;
- `direct_execute`.

Hai chế độ này nên để sau khi hệ thống đủ trưởng thành.

### 9.1.7. Scheduler

Hỗ trợ chạy agent theo lịch:

- hằng ngày;
- hằng tuần;
- cron expression;
- timezone;
- pause/resume schedule.

### 9.1.8. Sandbox Execution

Phiên bản 1.0 cần sandbox tối thiểu cho tool execution:

- giới hạn timeout;
- giới hạn memory;
- giới hạn network domain;
- không cho truy cập filesystem nếu không được cấp;
- không cho đọc secret ngoài phạm vi;
- log execution metadata.

### 9.1.9. Execution Trace

Mỗi lần chạy cần ghi:

- request;
- agent;
- plan;
- approval;
- tool call;
- tool result;
- output;
- status;
- error;
- cost;
- duration.

### 9.1.10. GUI Dashboard cơ bản

Tối thiểu có:

- danh sách agent;
- màn hình chi tiết agent;
- approval queue;
- execution history;
- tool registry;
- schedule list;
- pause/resume/kill action.

## 9.2. Nhóm tính năng nên có nếu đủ thời gian

- CLI Executive;
- Telegram Bot Connector;
- Memory đơn giản;
- Vector search cho knowledge base;
- Replay execution;
- Basic evaluation;
- Cost dashboard;
- Tool generation ở mức prototype.

## 9.3. Nhóm tính năng chưa nên làm trong 1.0

- Agent tự ghi trực tiếp database production;
- Agent tự chạy shell/deploy/restart service;
- Tool Forge tự động publish không qua duyệt;
- Mesh multi-agent phức tạp;
- Auto-scaling agent runtime phức tạp;
- Memory tự động ghi tri thức chính thức không qua kiểm soát.

---

# 10. Các bài toán có thể giải quyết nếu triển khai thành công

## 10.1. Báo cáo định kỳ

Ví dụ:

- báo cáo tỷ giá hằng ngày;
- báo cáo doanh thu;
- báo cáo tiến độ công việc;
- báo cáo số lượng hồ sơ;
- báo cáo lỗi hệ thống;
- báo cáo ticket tồn đọng.

Agent có thể lấy dữ liệu, tổng hợp, định dạng và gửi qua Telegram, email hoặc dashboard.

## 10.2. Trợ lý tạo dữ liệu import

Ví dụ:

- tổng hợp báo cáo công việc;
- chuẩn hóa danh sách khách hàng;
- tạo file import nhân sự;
- tạo file import danh mục;
- tạo dữ liệu migration;
- kiểm tra dữ liệu trước khi nhập hệ thống.

Agent không ghi trực tiếp vào DB, mà tạo file import hoặc staging data để con người kiểm tra.

## 10.3. Giám sát vận hành

Ví dụ:

- kiểm tra SSL certificate sắp hết hạn;
- kiểm tra dịch vụ chết;
- kiểm tra API chậm;
- tổng hợp lỗi log hằng ngày;
- cảnh báo disk đầy;
- theo dõi backup có chạy thành công không.

## 10.4. Trợ lý xử lý tài liệu

Ví dụ:

- tóm tắt tài liệu;
- phân loại tài liệu;
- trích xuất trường thông tin;
- so sánh hai phiên bản tài liệu;
- kiểm tra tài liệu có đủ biểu mẫu không;
- tạo checklist nghiệm thu.

## 10.5. Trợ lý kiểm tra tuân thủ

Ví dụ:

- kiểm tra hồ sơ có đủ thành phần không;
- kiểm tra tài liệu có theo mẫu chuẩn không;
- kiểm tra API contract có đủ field bắt buộc không;
- kiểm tra mã nguồn có thiếu test không;
- kiểm tra workflow có thiếu bước phê duyệt không.

## 10.6. Trợ lý kỹ thuật nội bộ

Ví dụ:

- tạo script hỗ trợ;
- đề xuất cấu hình;
- tạo file cấu hình mẫu;
- sinh lệnh kiểm tra hệ thống;
- tạo tài liệu hướng dẫn;
- kiểm tra log lỗi;
- tạo patch proposal.

Với nhóm này, mặc định nên dừng ở `proposal_only`, `draft_only` hoặc `generate_import_file`, chưa cho `direct_execute`.

---

# 11. Luồng hoạt động mẫu

## 11.1. Use case 1: Tạo agent báo cáo tỷ giá hằng ngày

### Yêu cầu người dùng

```text
Hàng ngày lúc 8h sáng, lấy báo cáo tỷ giá USD/VND, EUR/VND, JPY/VND và gửi vào Telegram cho tôi.
```

### Luồng xử lý

```text
1. Interaction Layer nhận yêu cầu.
2. Planner Agent phân tích intent: scheduled report.
3. Hệ thống chọn blueprint: Scheduled Report Agent.
4. Hệ thống kiểm tra Tool Registry:
   - exchange_rate_fetcher
   - telegram_sender
5. Nếu thiếu tool, Tool Forge đề xuất tạo tool.
6. Agent Governance đánh giá rủi ro: Level 1 + Level 2.
7. Hệ thống sinh CEO Approval Package.
8. Người quản lý phê duyệt.
9. Runtime chạy thử trong sandbox.
10. Người quản lý duyệt bật lịch.
11. Scheduler kích hoạt agent hằng ngày.
12. Agent lấy tỷ giá và gửi Telegram.
13. AgentOps ghi trace, cost, status.
```

### Agent Definition mẫu

```yaml
agent:
  name: daily_exchange_rate_reporter
  role: Chuyên viên báo cáo tỷ giá hằng ngày
  blueprint: Scheduled Report Agent

schedule:
  type: cron
  expression: "0 8 * * *"
  timezone: Asia/Ho_Chi_Minh

task:
  goal: Lấy tỷ giá USD/VND, EUR/VND, JPY/VND và gửi báo cáo Telegram

tools:
  - exchange_rate_fetcher
  - telegram_sender

action_mode:
  mode: send_notification

approval:
  required_before_first_run: true
  required_before_schedule_enabled: true

limits:
  max_runs_per_day: 1
  timeout_seconds: 60
  max_tool_calls_per_run: 5
```

---

## 11.2. Use case 2: Tạo file import báo cáo công việc hằng ngày

### Yêu cầu người dùng

```text
Hàng ngày tổng hợp báo cáo công việc từ các nguồn đầu vào và tạo file import để tôi kiểm tra trước khi đưa vào hệ thống.
```

### Luồng xử lý

```text
1. User gửi yêu cầu.
2. Planner Agent phân tích intent: data import assistant.
3. Chọn blueprint: Data Import Assistant.
4. Xác định nguồn dữ liệu cần đọc.
5. Xác định schema file import.
6. Sinh kế hoạch xử lý.
7. Trình người quản lý phê duyệt.
8. Agent chạy và tạo file import.
9. Hệ thống validate file.
10. Người quản lý kiểm tra.
11. Người quản lý import hoặc yêu cầu sửa.
12. Hệ thống ghi audit.
```

### Action Policy mẫu

```yaml
agent:
  name: daily_work_report_assistant

action_policy:
  mode: generate_import_file
  direct_database_write: false
  production_write: false
  require_human_review: true

output:
  type: import_file
  format: xlsx
  schema: daily_work_report_import_schema_v1

validation:
  schema_validation: true
  business_rule_validation: true
  duplicate_check: true
  required_fields_check: true

approval:
  approver_role: manager
  approval_required_before_import: true

audit:
  log_input_sources: true
  log_generated_file_hash: true
  log_validation_result: true
  log_approver: true
```

---

# 12. Hướng triển khai kỹ thuật

## 12.1. Kiến trúc dịch vụ đề xuất

Phiên bản đầu có thể triển khai theo kiến trúc modular monolith hoặc microservice nhẹ. Không nên tách quá nhiều service ngay từ đầu nếu đội ngũ nhỏ.

Đề xuất các module/service:

| Module/Service | Vai trò |
|---|---|
| `interaction-service` | API, GUI backend, webhook, Telegram connector |
| `approval-service` | Quản lý approval package và quyết định phê duyệt |
| `agent-registry-service` | Quản lý agent, blueprint, lifecycle |
| `orchestrator-service` | Điều phối workflow, scheduler, state machine |
| `agent-runtime-service` | Thực thi agent, gọi model, xử lý prompt contract |
| `tool-registry-service` | Quản lý tool contract, version, permission |
| `tool-runtime-service` | Chạy tool trong sandbox, gọi MCP/external tools |
| `memory-service` | Quản lý memory và knowledge base |
| `audit-service` | Ghi trace, metric, log, cost, replay data |
| `policy-service` | Kiểm tra permission, budget, action mode, risk level |

Nếu làm MVP, có thể gộp thành 3 khối:

```text
1. Control Plane
   - agent registry
   - approval
   - policy
   - dashboard API

2. Runtime Plane
   - orchestrator
   - agent runtime
   - tool runtime
   - scheduler

3. Data Plane
   - memory
   - audit
   - artifact storage
   - configuration store
```

## 12.2. Công nghệ đề xuất

### Backend

Có thể dùng một trong các hướng:

| Lựa chọn | Ưu điểm | Ghi chú |
|---|---|---|
| Java/Spring Boot | Phù hợp enterprise, mạnh về security, workflow, transaction | Phù hợp nếu muốn xây nền tảng doanh nghiệp nghiêm túc |
| Python/FastAPI | Nhanh cho AI/LLM/tooling, dễ tích hợp AI ecosystem | Phù hợp MVP nhanh |
| Go | Gọn, nhanh, tốt cho daemon/runtime/CLI | Phù hợp tool runtime hoặc control service |
| Node.js/NestJS | Phù hợp realtime dashboard, webhook, integration | Có thể dùng cho interaction layer |

Khuyến nghị thực tế:

```text
MVP: Python/FastAPI + React + PostgreSQL + Redis + Docker Sandbox
Enterprise hóa: Spring Boot/Go cho control plane, Python cho tool forge/runtime
```

### Frontend

- React hoặc Vue;
- Dashboard dạng admin console;
- WebSocket/SSE để theo dõi agent execution realtime.

### Database

- PostgreSQL: lưu metadata agent, tool, approval, audit index;
- Object Storage/S3/MinIO: lưu artifact, file import, output, log lớn;
- Vector Database: Qdrant, Milvus, pgvector hoặc OpenSearch vector;
- Redis: queue nhẹ, cache, runtime state tạm;
- Elasticsearch/OpenSearch: nếu cần audit/log search mạnh.

### Queue / Event Bus

- Redis Streams cho MVP;
- RabbitMQ hoặc Kafka nếu cần scale và event-driven rõ ràng.

### Sandbox

Tùy mức độ an toàn:

| Mức | Công nghệ | Ghi chú |
|---|---|---|
| Cơ bản | Docker container | Dễ triển khai MVP |
| Trung bình | gVisor / Kata Containers | Cách ly tốt hơn Docker thường |
| Cao | Firecracker microVM | Phù hợp chạy code không tin cậy |
| Cực chặt | Isolated worker pool + no network default | Phù hợp enterprise nhạy cảm |

### LLM Provider

Cần thiết kế Model Adapter để hỗ trợ nhiều model:

- OpenAI;
- Azure OpenAI;
- Google Gemini;
- Anthropic Claude;
- DeepSeek;
- local model qua Ollama/vLLM;
- model nội bộ.

Không nên gắn chặt hệ thống với một model duy nhất.

## 12.3. Agent Runtime

Agent Runtime cần có các thành phần:

```text
Prompt Contract
Model Adapter
Tool Call Controller
Memory Retriever
Output Validator
Policy Checker
Execution State
Retry/Timeout Handler
Trace Emitter
```

Luồng chạy agent:

```text
1. Load Agent Definition.
2. Load policy và permission.
3. Load memory scope.
4. Nhận task input.
5. Sinh plan hoặc thực thi theo workflow.
6. Trước mỗi tool call, kiểm tra policy.
7. Gọi tool runtime.
8. Validate output.
9. Sinh kết quả cuối.
10. Ghi execution trace.
```

## 12.4. Orchestrator

Orchestrator có thể triển khai bằng:

- state machine tự xây;
- workflow engine nhẹ;
- graph-based runtime;
- Temporal/Camunda nếu muốn workflow enterprise phức tạp.

Với MVP, nên dùng state machine hoặc graph đơn giản:

```text
REQUEST_RECEIVED
→ PLAN_GENERATED
→ WAITING_APPROVAL
→ APPROVED
→ SANDBOX_TESTING
→ READY_TO_RUN
→ RUNNING
→ COMPLETED / FAILED / PAUSED
```

## 12.5. Tool Forge

Tool Forge là phân hệ cho phép AI đề xuất hoặc sinh tool mới.

Pipeline đề xuất:

```text
Tool Need Detected
→ Search Tool Registry
→ Generate Tool Spec
→ Generate Code
→ Static Scan
→ Unit Test
→ Sandbox Test
→ Human Approval
→ Register Tool
→ Bind to Agent
```

Phiên bản 1.0 chỉ nên hỗ trợ Tool Forge ở mức kiểm soát chặt:

- sinh tool đơn giản;
- chỉ HTTP read-only hoặc transform dữ liệu;
- không sinh tool có quyền shell;
- không sinh tool ghi database production;
- bắt buộc approval trước khi register.

## 12.6. MCP Integration

MCP nên được dùng như chuẩn kết nối tool bên ngoài.

Vai trò:

- chuẩn hóa cách agent gọi tool;
- tách tool khỏi agent runtime;
- dễ mở rộng connector;
- giảm coupling giữa hệ thống và công cụ bên ngoài.

Trong Jarvis OS, MCP Client nằm trong Tool Runtime Layer. Tool có thể là MCP server độc lập hoặc adapter nội bộ.

---

# 13. Đề xuất kiến trúc dữ liệu

## 13.1. Agent Definition

```yaml
agent:
  id:
  name:
  role:
  blueprint_id:
  owner:
  created_by:
  approved_by:
  model:
  persona:
  status:
  action_mode:
  memory_scope:
  workspace_scope:
  schedule:
  budget:
  tools:
  policies:
  created_at:
  updated_at:
```

## 13.2. Tool Definition

```yaml
tool:
  id:
  name:
  version:
  description:
  input_schema:
  output_schema:
  permission_manifest:
  runtime_config:
  risk_level:
  owner:
  status:
  created_at:
  updated_at:
```

## 13.3. Approval Package

```yaml
approval_package:
  id:
  type: agent_creation | tool_registration | schedule_enable | permission_grant
  requester:
  generated_by_agent:
  summary:
  plan:
  risk_assessment:
  required_permissions:
  estimated_cost:
  action_mode:
  approval_status:
  approver:
  approved_at:
  rejection_reason:
```

## 13.4. Execution Trace

```yaml
execution_trace:
  id:
  agent_id:
  task_id:
  input:
  plan_id:
  approval_id:
  started_at:
  ended_at:
  status:
  tool_calls:
  output_artifacts:
  cost:
  latency_ms:
  error:
  policy_violations:
```

## 13.5. Memory Item

```yaml
memory_item:
  id:
  type:
  content:
  source:
  owner:
  scope:
  confidence:
  status:
  created_at:
  updated_at:
  expires_at:
  version:
```

Trạng thái memory:

```text
draft
verified
approved
deprecated
conflicted
deleted
```

---

# 14. Đề xuất kiến trúc bảo mật và kiểm soát

## 14.1. Phân quyền theo nguyên tắc tối thiểu

Agent chỉ được cấp quyền tối thiểu để hoàn thành nhiệm vụ.

Ví dụ agent báo cáo tỷ giá:

```text
Được gọi API tỷ giá đã whitelist.
Được gọi Telegram API.
Được đọc TELEGRAM_BOT_TOKEN.
Không được đọc database.
Không được ghi file tùy ý.
Không được chạy shell.
```

## 14.2. Phân loại rủi ro tool

| Level | Loại tool | Ví dụ | Cơ chế kiểm soát |
|---:|---|---|---|
| 0 | Pure function | format ngày, parse text | Có thể auto approve |
| 1 | Read-only public network | tỷ giá, thời tiết | Duyệt lần đầu |
| 2 | Notification tool | Telegram, email | Duyệt + giới hạn người nhận |
| 3 | Internal read | đọc DB, đọc file nội bộ | Duyệt + audit + scope |
| 4 | Internal write | ghi DB, tạo ticket | Duyệt workflow + staging |
| 5 | System execution | shell, deploy, xóa file | Cấm mặc định hoặc duyệt cực chặt |

## 14.3. Secret Management

Secret không được đưa trực tiếp vào prompt.

Nguyên tắc:

- secret lưu trong vault;
- agent không nhìn thấy secret raw nếu không cần;
- tool runtime inject secret khi thực thi;
- không log secret;
- secret scope theo tool và agent;
- có rotation policy.

## 14.4. Network Control

Tool runtime cần kiểm soát domain/IP được gọi:

```yaml
network:
  default: deny
  allow_domains:
    - api.telegram.org
    - api.exchangerate.host
```

## 14.5. Filesystem Control

Mặc định không cho tool đọc/ghi filesystem trừ vùng được cấp:

```yaml
filesystem:
  read:
    - /workspace/input
  write:
    - /workspace/output
```

## 14.6. Database Control

Không cấp quyền DB trực tiếp cho agent. Nếu cần, agent gọi tool đã kiểm soát.

Ví dụ:

```text
Agent → controlled_db_query_tool → read-only query đã giới hạn
```

Với ghi dữ liệu:

```text
Agent → staging_writer_tool → staging table → human approval → promote
```

---

# 15. AgentOps, giám sát và kiểm toán

AgentOps là lớp giúp doanh nghiệp biết hệ thống agent đang làm gì, tốn bao nhiêu, lỗi ở đâu và có vượt quyền không.

## 15.1. Metrics cần theo dõi

- số lần chạy agent;
- tỷ lệ thành công/thất bại;
- thời gian chạy;
- token sử dụng;
- chi phí theo agent;
- số tool call;
- lỗi theo loại;
- số lần policy violation;
- số approval pending;
- số agent đang active/paused/failed.

## 15.2. Trace cần lưu

- Decision Trace;
- Execution Trace;
- Tool Call Trace;
- Approval Trace;
- Artifact Trace;
- Memory Access Trace.

## 15.3. Replay

Replay cho phép xem lại một lần chạy:

```text
Input là gì?
Agent lập kế hoạch thế nào?
Ai duyệt?
Tool nào được gọi?
Output là gì?
Sai ở bước nào?
Có thể chạy lại không?
```

## 15.4. Circuit Breaker

Cần có cơ chế dừng tự động khi:

- agent gọi tool quá nhiều;
- lặp cùng một hành động;
- vượt token budget;
- timeout;
- lỗi liên tục;
- gọi tool bị từ chối nhiều lần;
- phát hiện rủi ro policy.

---

# 16. Những khó khăn kỹ thuật cần giải quyết

## 16.1. Kiểm soát tool do AI sinh ra

Đây là vấn đề khó nhất.

Rủi ro:

- code sinh ra có lỗi;
- code gọi domain không được phép;
- code làm lộ secret;
- code ghi file sai chỗ;
- code tạo request nguy hiểm;
- code bị prompt injection dẫn hướng.

Giải pháp cần có:

- Tool Contract bắt buộc;
- static scan;
- unit test;
- sandbox execution;
- network allowlist;
- secret isolation;
- human approval;
- versioning;
- rollback.

## 16.2. Đảm bảo agent không vượt quyền

Agent có thể cố gọi tool ngoài quyền hoặc sinh tham số nguy hiểm.

Cần có Policy Enforcement Point trước mỗi tool call:

```text
Agent muốn gọi tool
→ Policy kiểm tra quyền
→ Tool Runtime kiểm tra manifest
→ Sandbox thực thi
→ Audit ghi lại
```

Không được tin vào lời agent nói rằng nó “sẽ không làm gì nguy hiểm”.

## 16.3. Quản lý memory không bị ô nhiễm

Memory có thể bị sai, lỗi thời hoặc mâu thuẫn.

Cần giải quyết:

- memory nào được ghi tự động;
- memory nào cần duyệt;
- memory nào hết hạn;
- memory nào bị conflict;
- memory nào là tri thức chính thức;
- memory nào chỉ là ghi chú tạm.

## 16.4. Kiểm soát chi phí token

Multi-agent rất dễ tốn token.

Cần có:

- max token per run;
- max cost per agent per day/month;
- max iterations;
- max tool calls;
- summarization policy;
- context pruning;
- model routing: việc nhẹ dùng model rẻ, việc khó dùng model mạnh.

## 16.5. Xử lý prompt injection

Agent có thể đọc dữ liệu từ web, email, tài liệu hoặc file. Trong đó có thể chứa chỉ dẫn độc hại.

Cần có:

- phân tách system instruction và untrusted content;
- tool permission độc lập với nội dung đọc được;
- không cho nội dung bên ngoài tự thay đổi policy;
- kiểm tra output trước khi action;
- human approval với action rủi ro.

## 16.6. Độ tin cậy của output

LLM có thể sai.

Cần có:

- output schema validation;
- business rule validation;
- deterministic check;
- confidence score;
- source citation;
- human review;
- dry-run mode;
- staging/import file thay vì commit trực tiếp.

## 16.7. Điều phối nhiều agent

Multi-agent dễ tạo ra phức tạp:

- agent tranh quyền;
- lặp trao đổi vô ích;
- chi phí tăng;
- khó debug;
- không rõ ai chịu trách nhiệm.

Trong 1.0 nên hạn chế multi-agent mesh phức tạp. Ưu tiên sequential hoặc hierarchical workflow rõ ràng.

## 16.8. Sandbox đủ an toàn

Docker thường chưa đủ nếu chạy code không tin cậy.

Cần đánh giá:

- container escape;
- network isolation;
- secret leakage;
- resource exhaustion;
- filesystem isolation;
- cleanup sau khi chạy.

MVP có thể dùng Docker nhưng enterprise nên nghiên cứu gVisor, Kata Containers hoặc Firecracker.

## 16.9. Thiết kế UX phê duyệt

Nếu bắt người dùng duyệt quá nhiều, hệ thống sẽ khó dùng. Nếu duyệt quá ít, hệ thống rủi ro.

Cần thiết kế approval theo risk level:

- việc an toàn có thể auto approve;
- việc trung bình duyệt lần đầu;
- việc nguy hiểm duyệt từng lần;
- việc rất nguy hiểm cấm mặc định.

## 16.10. Khả năng replay và audit đầy đủ

Nếu không lưu đủ trace, khi agent sai sẽ không biết sai ở đâu.

Cần thiết kế audit ngay từ đầu, không để sau.

---

# 17. Lộ trình triển khai đề xuất

## 17.1. Giai đoạn 1 — Core Runtime có kiểm soát

Mục tiêu:

- tạo agent thủ công hoặc bán tự động;
- có tool registry;
- có approval package;
- chạy được agent đơn giản;
- có execution trace.

Thành phần:

```text
Agent Registry
Tool Registry
Approval Service
Policy Service
Agent Runtime
Tool Runtime
Basic Dashboard
Execution Trace
```

## 17.2. Giai đoạn 2 — Agent Blueprint và scheduled agent

Mục tiêu:

- người dùng mô tả nhu cầu;
- hệ thống tạo agent từ blueprint;
- chạy agent theo lịch.

Thành phần:

```text
Scheduled Report Agent Blueprint
Monitoring Agent Blueprint
Data Import Assistant Blueprint
Scheduler
Approval Queue
Telegram Connector
File Artifact Store
```

Use case demo:

```text
Tạo agent báo cáo tỷ giá hằng ngày và gửi Telegram.
```

## 17.3. Giai đoạn 3 — Action Buffer

Mục tiêu:

- agent tạo file import;
- agent ghi staging;
- người quản lý duyệt trước khi commit.

Thành phần:

```text
Import File Generator
Staging Writer
Validation Engine
Approval Workflow
Artifact Versioning
```

Use case demo:

```text
Agent tổng hợp báo cáo công việc hằng ngày và tạo file import.
```

## 17.4. Giai đoạn 4 — Tool Forge có kiểm soát

Mục tiêu:

- AI đề xuất tool mới;
- AI sinh tool đơn giản;
- tool được test, sandbox, approval rồi mới đăng ký.

Thành phần:

```text
Tool Spec Generator
Code Generator
Static Scan
Unit Test Runner
Sandbox Test
Tool Approval
Tool Versioning
```

## 17.5. Giai đoạn 5 — AgentOps nâng cao

Mục tiêu:

- dashboard vận hành đầy đủ;
- cost tracking;
- replay;
- evaluation;
- memory governance.

Thành phần:

```text
Cost Dashboard
Replay Viewer
Failure Analysis
Memory Management
Policy Violation Alert
Agent Performance Report
```

---

# 18. Kết luận

Jarvis OS là một ý tưởng có tính khả thi kỹ thuật và có giá trị thực tế nếu được triển khai theo hướng nền tảng quản trị agent cấp doanh nghiệp.

Điểm quan trọng nhất không phải là tạo thật nhiều agent, mà là tạo ra một cơ chế để doanh nghiệp có thể biến nhu cầu tự nhiên thành năng lực vận hành mới:

```text
Người dùng mô tả nhu cầu.
AI tạo agent/tool/workflow.
Hệ thống đánh giá rủi ro.
Con người phê duyệt.
Agent vận hành trong phạm vi được kiểm soát.
Mọi hành động đều có trace, audit và khả năng can thiệp.
```

Tinh thần cốt lõi có thể tóm lại bằng 5 nguyên tắc:

```text
1. AI được quyền đề xuất và tạo năng lực mới.
2. Agent/tool không được tự cấp quyền cho chính mình.
3. Không phải hành động nào cũng ghi trực tiếp vào hệ thống lõi.
4. Draft, import file, staging và approval queue là vùng đệm bắt buộc cho tác vụ rủi ro.
5. Con người giữ quyền cam kết cuối cùng đối với dữ liệu và hành động quan trọng.
```

Nếu triển khai thành công, Jarvis OS có thể trở thành một lớp vận hành mới trong doanh nghiệp: nơi con người không cần tự viết từng script, từng job, từng automation nhỏ; thay vào đó họ mô tả nhu cầu, còn AI giúp kiến tạo năng lực vận hành mới dưới sự kiểm soát chặt chẽ của kiến trúc, chính sách và con người.

---

# Phụ lục A — Ví dụ CEO Approval Package

```markdown
# Tờ trình tạo Agent mới

## 1. Tên agent
Daily Exchange Rate Reporter

## 2. Mục tiêu
Hằng ngày lấy tỷ giá USD/VND, EUR/VND, JPY/VND và gửi báo cáo vào Telegram.

## 3. Lịch chạy
08:00 hằng ngày, múi giờ Asia/Ho_Chi_Minh.

## 4. Công cụ sử dụng
- exchange_rate_fetcher: gọi API tỷ giá công khai.
- telegram_sender: gửi tin nhắn Telegram.

## 5. Quyền cần cấp
- Được gọi API tỷ giá đã whitelist.
- Được gọi Telegram Bot API.
- Được đọc secret TELEGRAM_BOT_TOKEN.
- Không được đọc/ghi file ngoài workspace.
- Không được truy cập database.
- Không được chạy shell.

## 6. Action Mode
send_notification

## 7. Chi phí dự kiến
- 1 lần chạy/ngày.
- Token thấp.
- Network call thấp.

## 8. Rủi ro
- API tỷ giá lỗi.
- Telegram chat_id cấu hình sai.
- Báo cáo sai định dạng.

## 9. Kiểm soát
- Chạy thử trước khi bật lịch.
- Không log secret.
- Có thể pause agent bất cứ lúc nào.
- Ghi audit toàn bộ lần chạy.
```

---

# Phụ lục B — Ví dụ trạng thái vòng đời Agent

```text
proposed
→ approved
→ deployed
→ active
→ paused
→ deprecated
→ archived
```

---

# Phụ lục C — Ví dụ trạng thái Tool

```text
draft
→ generated
→ tested
→ pending_approval
→ approved
→ registered
→ deprecated
→ revoked
```

---

# Phụ lục D — Ví dụ trạng thái Memory

```text
draft
→ verified
→ approved
→ deprecated
→ conflicted
→ deleted
```
