# DANH MỤC TOÀN BỘ CHỨC NĂNG & MAPPING HỆ THỐNG SAT-MOTOR

Tài liệu này cung cấp bản đồ chi tiết của **tất cả** các chức năng, menu con, bước nghiệp vụ, form ẩn/tích hợp, các API backend dịch vụ được gọi (BE Service calls) và toàn bộ 43 bảng cơ sở dữ liệu (Database Tables) liên quan trực tiếp trong luồng xử lý của phân hệ **sat-motor**.

---

> [!NOTE]
> Phân hệ `sat-motor` là thành phần cốt lõi trong hệ thống bảo hiểm xe cơ giới, kết nối chặt chẽ giữa giao diện Angular hiện đại, cổng API Gateway trung gian và các cơ sở dữ liệu nghiệp vụ (Core DB và Portal DB).

---

## 1. DANH MỤC TOÀN BỘ PHÂN HỆ GIAO DIỆN (FRONTEND - `sat-motor`)
Frontend Angular quản lý giao diện thông qua hai file cấu hình tuyến đường chính: `app.routes.ts` (các trang dùng chung, xác thực và cấu hình cổng) và `motor.routes.ts` (các luồng tính toán bảo hiểm chi tiết).

```
sat-motor (Views Directory Structure)
 ├── enquiry/                       --> Phân hệ tra cứu & báo cáo
 ├── home/                          --> Trang chủ Alpha Portal
 ├── not-found/                     --> Trang báo lỗi 404
 └── motor/                         --> Phân hệ xe cơ giới chính
      ├── issuance/                 --> Quy trình Cấp đơn mới (Cover Note)
      └── endorsement/              --> Quy trình Sửa đổi bổ sung (Endorsement)
```

---

## 2. CHI TIẾT CÁC CHỨC NĂNG & MAPPING NGHIỆP VỤ

### A. Phân hệ Cấp mới đơn bảo hiểm (Cover Note Issuance)
Tuyến đường chính: `/views/motor/issuance`. Quy trình gồm 7 bước liên tục (Stepper).

| STT | Bước nghiệp vụ (Tab/Step) | Component / Form tương ứng | Mô tả chức năng & các form ẩn/tích hợp bên trong | BE Service Call (API & Endpoint code) | Database Tables (Đọc/Ghi trong luồng) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **Chọn Đại lý**<br>*(Agent Selection)* | `AgentSelectionComponent` | Chọn mã đại lý thao tác đơn bảo hiểm. Kiểm tra quyền hạn đại lý. | **`sat-bo-services`**<br>- `BO_AGENTS`<br>- `BO_AGENT_DETAIL` | **Đọc:**<br>- `SASC_USER`<br>- `MKAG_AGENT` |
| **2** | **Thông tin bổ sung đại lý**<br>*(Agent Info)* | `AgentAdditionalInfoComponent` | Đồng bộ dữ liệu Unified Dashboard (`agent-information`, `agents-information`) và lấy thông tin tài khoản CBC. | **`sat-bo-services`** / **`alpha-sat-services`**<br>- `BO_AGENT_CBC_INFO` / `ALPHA_AGENT_CBC_INFO`<br>- `BO_AGENT_CNOTE_STOCK` | **Đọc:**<br>- `MKAG_AGENT_DEALER`<br>- `SAPM_CONFIG` |
| **3** | **Khai báo thông tin chính**<br>*(Declaration & Vehicle)* | `DeclarationIssuanceInfoComponent` & `issuance.component.ts` | **Bước nhập liệu cốt lõi chứa nhiều thành phần ẩn:**<br>1. **Vehicle Search**: Tra cứu thông tin xe dựa trên biển số xe (`vehicle-details`).<br>2. **NVIC Model Lookup**: Tra cứu hãng, mẫu và đời xe từ mã NVIC (`CMUW_NVIC`).<br>3. **Sum Insured Range**: Thanh trượt chọn khoảng giá trị bảo hiểm đề xuất (`vehicle-sum-insured`).<br>4. **PDPA Consent**: Form xác thực đồng ý điều khoản bảo vệ dữ liệu cá nhân (`pdpa`).<br>5. **eDoc Consent**: Form đồng ý nhận chứng từ điện tử (`consent-indicator`).<br>6. **Blacklist Check**: Tự động kiểm duyệt danh sách đen của chủ xe/phương tiện. | **`sat-issuance-services`** & **`sat-bo-services`**<br>- `ISS_CN_SCH_PRE_INFO`<br>- `ISS_CHK_BLACKLIST`<br>- `BO_SEARCH_NIVC`<br>- `BO_PRODUCT_CONFIGURATION`<br>- `ISS_CN_ENQUIRE_VIX` | **Đọc:**<br>- `CNGE_NOTE`<br>- `CNGE_NOTE_MT`<br>- `CNGE_RISK_VEHICLE`<br>- `CMUW_NVIC`<br>- `CMUW_MODEL_VEH_INVIC` (Thực tế: `CMUW_MODEL_VEH_NVIC`) <br>- `CMGC_CODE_DET`<br>- `SAPM_PRODUCT_CONFIG`<br>**Ghi:**<br>- `CNMT_NOTE_ISM` |
| **4** | **Cấu hình quyền lợi**<br>*(Coverage Selection)* | `CoverageComponent` & `coverages` | **Chọn gói và quyền lợi phụ thu:**<br>1. **Basic Premium**: Tính toán phí bảo hiểm cơ bản dựa trên loại xe, khu vực rủi ro địa lý.<br>2. **Extra Cover / Add-ons**: Form chọn mua thêm quyền lợi bảo hiểm bổ sung (Windscreen, Flood...).<br>3. **Thailand Trip**: Form mở rộng bảo hiểm khi lái xe du lịch sang Thái Lan (Cross-border).<br>4. **E-hailing Driver**: Bảo hiểm phụ thu dành cho tài xế xe công nghệ (`e-hailing-driver`).<br>5. **Named Drivers**: Form thêm danh sách tối đa 4 người lái xe chỉ định (`named-driver`, `driver-name`). | **`sat-issuance-services`** & **`sat-premium-services`** & **`sat-bo-services`**<br>- `ISS_CN_PREM_CALC` / `SAT_PREMIUM_CALC`<br>- `BO_EXTRA_COVER` / `BO_EXTRA_COVER_ALL`<br>- `ISS_CN_PRODUCT_VALIDATION` | **Đọc:**<br>- `SAPM_PRODUCT_CONFIG`<br>- `CMUW_MTVEHUSE`<br>- `CMUW_MTVEHUSE_NCD`<br>**Ghi (dự kiến):**<br>- `CNGE_COVER`<br>- `CNGE_COVER_MT`<br>- `CNGE_SUBCOVER`<br>- `CNGE_RISK_DRVR`<br>- `CNGE_COVER_EXCESS`<br>- `CNGE_COVER_LOADING` |
| **5** | **Hồ sơ khách hàng**<br>*(Customer Partner)* | `CustomerPartnerComponent` & `customer-partner` | **Form nhập liệu thông tin chủ sở hữu đơn bảo hiểm:**<br>1. **Client Type**: Lựa chọn Khách hàng cá nhân (Individual) hoặc Doanh nghiệp (Company).<br>2. **E-Invoicing details**: Nhập mã số thuế cá nhân hoặc doanh nghiệp để xuất E-Invoice theo chuẩn IRB (`e-invoice-individual`, `e-invoice-company`).<br>3. **Geolocation conversion**: Chuẩn hóa địa chỉ nhập thành toạ độ địa lý và Plus Code (`geolocation`). | **`sat-cp-services`** / **`alpha-cp-services`**<br>- `CP_DETAIL` / `CP_SAVE`<br>- `CP_VALIDATE` / `CP_VALIDATE_AGE`<br>- `CP_CONVERT_GEO_FORMATTED_ADDRESS` | **Đọc/Ghi:**<br>- `MKCL_PARTNER`<br>- `MKCL_PARTNER_ADDRESS`<br>- `MKCL_PARTNER_AGENT` |
| **6** | **Tóm tắt tính phí**<br>*(Summary & Premium)* | `SummaryComponent` & `premium-summary` | **Tổng kết, điều chỉnh và duyệt cấp đơn:**<br>1. **Premium Information**: Hiển thị bảng phí chi tiết gồm phí gốc, chiết khấu NCD, thuế dịch vụ ST, phí tem Stamp duty (`premium-information`).<br>2. **NCD Enquiry & Confirm**: Hiển thị trạng thái chiết khấu NCD đã xác minh từ ISM (`no-claim-discount`, `ncd-confirmation`).<br>3. **Refer Case Workflow**: Form gửi yêu cầu quản trị viên phê duyệt đơn vi phạm quy tắc hệ thống (`refer-case`, `referral-details`).<br>4. **Generate Quotation**: Nút bấm tạo báo giá gửi khách hàng.<br>5. **Cancel Draft**: Nút huỷ bỏ đơn nháp (`request-cancellation`). | **`sat-issuance-services`**<br>- `ISS_CN_SAVE_DRAFT`<br>- `ISS_CN_GENERATE_QUOTATION`<br>- `ISS_GET_NCD_ENQUIRY`<br>- `ISS_NCD_CONFIRMATION_ENQUIRY`<br>- `ISS_CN_APPROVAL` / `ISS_CN_REJECT_APPROVAL`<br>- `ISS_VALIDATE_QUOTATION_CUTOFF_DATE`<br>- `ISS_CN_CANCEL_DRAFT` | **Đọc/Ghi (Chính thức):**<br>- `CNGE_NOTE`<br>- `CNGE_NOTE_MT`<br>- `CNGE_RISK`<br>- `CNGE_RISK_VEHICLE`<br>- `CNGE_COVER`<br>- `CNGE_COVER_MT`<br>- `CNGE_SUBCOVER`<br>- `CNGE_RISK_DRVR`<br>- `CNGE_COVER_EXCESS`<br>- `CNGE_COVER_LOADING`<br>- `CNGE_COVER_NCD_INFO`<br>- `CNGE_NOTE_STAX`<br>- `CNGE_NOTE_REFER`<br>- `CNGE_NOTE_REFER_MANUAL`<br>- `CNGE_NOTE_REFER_REMARKS`<br>- `SAPM_DOC_NUM_SEQGEN`<br>- `CNMT_NOTE_ISM` |
| **7** | **Hoàn tất cấp đơn**<br>*(Complete)* | `CompleteComponent` & `complete-information` | **Màn hình hiển thị kết quả cuối cùng:**<br>1. **Send to JPJ**: Gửi tín hiệu cấp chứng thư và thuế đường bộ lên JPJ (`send-to-jpj`).<br>2. **Print/Download Documents**: Cho phép tải về tài liệu Cover Note, Certificate of Insurance, Schedule, Receipt. | **`sat-issuance-services`** & **`sat-report-services`** & **`sat-communication-services`**<br>- `ISS_CN_SEND_JPJ`<br>- `ISS_CN_ISM_UPDATE`<br>- `SAT_REP_CVN`<br>- `SAT_COM_EMAIL` | **Ghi/Cập nhật:**<br>- `CNGE_NOTE` (đổi trạng thái đơn thành `IN-FORCE`)<br>- `CNGE_NOTE_PYMT`<br>- `CNMT_NOTE_JPJ`<br>- `CNMT_NOTE_ISM`<br>- `CNGE_NOTE_DOC` (Thực tế: `CNGE_DOC`) |

---

### B. Phân hệ Sửa đổi bổ sung đơn bảo hiểm (Endorsement)
Tuyến đường chính: `/views/motor/endorsement`. Dành cho các hợp đồng đang hiệu lực cần thay đổi thông tin:

*   **Tìm kiếm hợp đồng cần sửa**: Tích hợp màn hình tra cứu số hợp đồng gốc (`HTGE_POL`).
    *   *BE Service Call*: **`sat-issuance-services`** $\rightarrow$ `ISS_ENDT_SEARCH_POLICY`, `ISS_POL_SEARCH_DETAIL`.
    *   *Database Tables*: **Đọc:** `HTGE_POL`, `HTGE_RISK`, `HTGE_RISK_VEHICLE`, `HTGE_COVER`, `HTGE_SUBCOVER` (lấy dữ liệu đơn gốc).
*   **Khai báo thay đổi (Endorsement Details)**: Sử dụng `EndorsementDetailsComponent`.
    *   *Form ẩn thay đổi kết cấu xe*: Thay đổi dung tích động cơ, số chỗ ngồi, tải trọng.
    *   *Form ẩn thay đổi chủ quyền*: Đổi tên chủ xe, đổi biển số xe tạm thời sang biển số chính thức.
    *   *Endorsement Narration*: Trình biên tập tích hợp soạn thảo nội dung diễn giải của sửa đổi bổ sung (`endorsement-narration`).
    *   *BE Service Call*: **`sat-issuance-services`** $\rightarrow$ `ISS_ENDT_ENQUIRE_NARRATION`, `ISS_ENDT_VALIDATION`.
    *   *Database Tables*: **Ghi (nháp):** `CNGE_NOTE_WIP`.
*   **Tính toán lại phí chênh lệch**: Tính phí bổ sung (Debit Note) hoặc hoàn phí (Credit Note) tại `EndorsementSummaryComponent`.
    *   *BE Service Call*: **`sat-issuance-services`** $\rightarrow$ `ISS_ENDT_CALC`.
    *   *Database Tables*: **Đọc/Ghi:** `CNGE_NOTE_WIP`, `CNGE_COVER_MT`.
*   **Hoàn tất Endorsement**: Gửi thông tin JPJ cập nhật và xuất bản tài liệu Endorsement (`EndorsementCompleteComponent`).
    *   *BE Service Call*: **`sat-issuance-services`** $\rightarrow$ `ISS_ENDT_SEND_JPJ`, `ISS_ENDT_GENERATE_QUOTATION`.
    *   *Database Tables*: **Ghi:** `REVISION_CNGE_NOTE` (Thực tế: `REV_CNGE_NOTE`), `REVISION_CNGE_NOTE_MT` (Thực tế: `REV_CNGE_NOTE_MT`), cập nhật `CNGE_NOTE`, `CNMT_NOTE_JPJ`.

---

### C. Phân hệ Tra cứu, Báo cáo & Phê duyệt (Enquiry & Reports)
Nhóm chức năng hỗ trợ nghiệp vụ và quản trị được định tuyến qua `/views/enquiry`:

1.  **Tra cứu Báo giá (View Quotation)** (`ViewQuotationComponent`): Tìm kiếm, kiểm tra trạng thái và sửa đổi các bản báo giá của xe.
    *   *BE Service Call*: **`sat-issuance-services`** $\rightarrow$ `ISS_OPP_SEARCH_OCC`, `ALPHA_OPP_SEARCH_OCC`.
    *   *Database Tables*: **Đọc/Cập nhật:** `CNGE_NOTE`, `CNGE_RISK_VEHICLE`.
2.  **Tra cứu Hợp đồng gốc (View Policy)** (`ViewPolicyComponent`): Tra cứu thông tin chi tiết các đơn bảo hiểm đã cấp chính thức.
    *   *BE Service Call*: **`sat-issuance-services`** $\rightarrow$ `ISS_POL_SEARCH_CRITERIA`, `ISS_HTGEPOL_SEARCH_DETAIL`.
    *   *Database Tables*: **Đọc:** `HTGE_POL`, `HTGE_RISK_VEHICLE`, `MKCL_PARTNER`.
3.  **Tra cứu Sửa đổi bổ sung (View Endorsement)** (`ViewEndorsementComponent`): Tìm kiếm lịch sử sửa đổi, bổ sung của xe.
    *   *BE Service Call*: **`sat-issuance-services`** $\rightarrow$ `ISS_ENDT_LISTING`.
    *   *Database Tables*: **Đọc:** `REVISION_CNGE_NOTE` (Thực tế: `REV_CNGE_NOTE`), `REVISION_CNGE_NOTE_MT` (Thực tế: `REV_CNGE_NOTE_MT`).
4.  **Phê duyệt đơn Referral (View Approval)** (`ViewApprovalComponent`): Màn hình quản trị dành cho Manager duyệt hoặc bác bỏ các đơn bảo hiểm bị chặn do vi phạm quy tắc nghiệp vụ.
    *   *BE Service Call*: **`sat-issuance-services`** $\rightarrow$ `ISS_CN_APPROVAL_DETAIL`, `ISS_CN_APPROVAL`.
    *   *Database Tables*: **Đọc/Cập nhật:** `CNGE_NOTE`, `CNGE_NOTE_REFER`, `CNGE_NOTE_REFER_MANUAL`, `CNGE_NOTE_REFER_REMARKS`.
5.  **Tải lên dữ liệu hàng loạt (Endorsement Data Upload)** (`EndorsementDataUploadComponent`): Cho phép import danh sách sửa đổi hàng loạt từ file Excel.
    *   *BE Service Call*: **`sat-doc-services`** $\rightarrow$ `SAT_DOC_UPLOAD`, `SAT_DOC_COMMIT`.
    *   *Database Tables*: **Ghi:** `CNGE_NOTE_DOC` (Thực tế: `CNGE_DOC`).
6.  **Lịch sử yêu cầu chi tiết (CN & Endorsement History)**: Bảng nhúng hiển thị dòng thời gian giao dịch của phương tiện (`cn-history-table`, `endt-history-table`).
    *   *Database Tables*: **Đọc:** `CNGE_NOTE`, `REVISION_CNGE_NOTE` (Thực tế: `REV_CNGE_NOTE`).
7.  **Tra cứu lịch sử khiếu nại (Historical Claims)** (`historical-claim`, `claim-details-table`): Tra cứu lịch sử tai nạn, đền bù bảo hiểm của xe trong quá khứ.
    *   *BE Service Call*: **`sat-issuance-services`** $\rightarrow$ `ISS_HTGEPOL_SEARCH_CLAIM`.
    *   *Database Tables*: **Đọc:** `CLAIM_DETAILS` / `HISTORICAL_CLAIM` (Thực tế: `HTCL_CLAIM`, `HTCL_CLAIM_DET`, `CLMT_MAST`).

---

## 3. ĐỐI CHIẾU VỚI DANH SÁCH BẢNG VẬT LÝ VÀ TRẠNG THÁI TRÊN POSTGRESQL (PG)

Dưới đây là bảng đối chiếu chi tiết 43 bảng nghiệp vụ so với danh sách bảng vật lý và trạng thái tồn tại trên hệ quản trị cơ sở dữ liệu **PostgreSQL (Exists in PG?)** dựa trên dữ liệu di trú hệ thống:

| STT | Logical Table (Doc) | Physical Table (DB) | Schema | Data Row ID | Exists in PG? | Description (Migration Catalog) | Modules / Microservices |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **1** | `CNGE_NOTE` | `CNGE_NOTE` | `ALPHADB` | **241** | **Yes** | Table contains cover note master info / General Cover Note metadata | `sat-inssurace-service`, `sat-motor` (FE) |
| **2** | `CNGE_NOTE_MT` | `CNGE_NOTE_MT` | `ALPHADB` | **254** | **Yes** | Table to store motor vehicle details associated with the Cover Note | `sat-inssurace-service`, `sat-motor` (FE) |
| **3** | `CNGE_RISK` | `CNGE_RISK` | `ALPHADB` | **269** | **Yes** | Table contains cover note risk details / Risk metadata | `sat-inssurace-service`, `sat-motor` (FE) |
| **4** | `CNGE_RISK_VEHICLE` | `CNGE_RISK_VEHICLE` | `ALPHADB` | **283** | **Yes** | Table to store detailed risk characteristics of the vehicle | `sat-inssurace-service`, `sat-motor` (FE) |
| **5** | `CNGE_COVER` | `CNGE_COVER` | `ALPHADB` | **215** | **Yes** | Table to store cover details / Scope of coverage | `sat-inssurace-service`, `sat-motor` (FE) |
| **6** | `CNGE_COVER_MT` | `CNGE_COVER_MT` | `ALPHADB` | **230** | **Yes** | Table to store coverage plans and add-on codes for motor vehicles | `sat-inssurace-service`, `sat-motor` (FE) |
| **7** | `CNGE_SUBCOVER` | `CNGE_SUBCOVER` | `ALPHADB` | **284** | **Yes** | Table to store subcovers / optional add-ons (e.g. windscreen, flood) | `sat-inssurace-service`, `sat-motor` (FE) |
| **8** | `CNGE_RISK_DRVR` | `CNGE_RISK_DRVR` | `ALPHADB` | **274** | **Yes** | Table to store designated driver details (Named Drivers - max 4) | `sat-inssurace-service`, `sat-motor` (FE) |
| **9** | `CNGE_COVER_EXCESS` | `CNGE_COVER_EXCESS` | `ALPHADB` | **220** | **Yes** | Table to store deductibles and excess details for coverages | `sat-inssurace-service`, `sat-motor` (FE) |
| **10** | `CNGE_COVER_LOADING` | `CNGE_COVER_LOADING` | `ALPHADB` | **226** | **Yes** | Table to store loading fees / premium loading adjustments | `sat-inssurace-service`, `sat-motor` (FE) |
| **11** | `CNGE_COVER_NCD_INFO` | `CNGE_COVER_NCD_INFO` | `ALPHADB` | **231** | **Yes** | Table contains Motor Quotation NCD (No Claim Discount) from Radar Live | `sat-inssurace-service`, `sat-motor` (FE) |
| **12** | `CNGE_NOTE_STAX` | `CNGE_NOTE_STAX` | `ALPHADB` | **265** | **Yes** | Table to store policy-level Service Tax breakdowns | `sat-inssurace-service`, `sat-motor` (FE) |
| **13** | `CNGE_NOTE_PYMT` | `CNGE_NOTE_PYMT` | `ALPHADB` | **259** | **Yes** | Table to store transaction and payment details of the policy | `sat-inssurace-service`, `sat-motor` (FE) |
| **14** | `CNGE_NOTE_REFER` | `CNGE_NOTE_REFER` | `ALPHADB` | **260** | **Yes** | Table to store logs and status of referred cover notes (Referral cases) | `sat-inssurace-service`, `sat-motor` (FE) |
| **15** | `CNGE_NOTE_REFER_MANUAL` | `CNGE_NOTE_REFER_MANUAL` | `ALPHADB` | **261** | **Yes** | Table to store manual underwriting referral configurations / rules | `sat-inssurace-service`, `sat-motor` (FE) |
| **16** | `CNGE_NOTE_REFER_REMARKS` | `CNGE_NOTE_REFER_REMARKS` | `ALPHADB` | **262** | **Yes** | Table to store referral review comments and approval remarks | `sat-inssurace-service`, `sat-motor` (FE) |
| **17** | `CNGE_NOTE_DOC` | **`CNGE_DOC`** | `ALPHADB` | **236** | **Yes** | Table contains uploaded file metadata linked with FIS (File Integration System) | `sat-doc-service`, `sat-inssurace-service`, `sat-motor` (FE) |
| **18** | `CNGE_NOTE_WIP` | `CNGE_NOTE_WIP` | `ALPHADB` | **267** | **Yes** | Table to store draft transactions for active Endorsements (WIP details) | `sat-inssurace-service`, `sat-motor` (FE) |
| **19** | `CNMT_NOTE_ISM` | `CNMT_NOTE_ISM` | `ALPHADB` | **324** | **Yes** | Table to store ISM integration log / ISM sync status | `sat-inssurace-service`, `sat-motor` (FE) |
| **20** | `CNMT_NOTE_JPJ` | `CNMT_NOTE_JPJ` | `ALPHADB` | **327** | **Yes** | Table to store JPJ integration log / JPJ sync status | `sat-inssurace-service`, `sat-motor` (FE) |
| **21** | `HTGE_POL` | `HTGE_POL` | `ALPHADB` | **422** | **Yes** | Table contains official policy information (Post-Issuance Core Policy) | `sat-inssurace-service`, `sat-motor` (FE) |
| **22** | `HTGE_RISK` | `HTGE_RISK` | `ALPHADB` | **433** | **Yes** | Table contains historical policy risk metadata | `sat-inssurace-service`, `sat-motor` (FE) |
| **23** | `HTGE_RISK_VEHICLE` | `HTGE_RISK_VEHICLE` | `ALPHADB` | **441** | **Yes** | Table contains historical vehicle risk specifications | `sat-inssurace-service`, `sat-motor` (FE) |
| **24** | `HTGE_COVER` | `HTGE_COVER` | `ALPHADB` | **402** | **Yes** | Table contains historical policy cover details | `sat-inssurace-service`, `sat-motor` (FE) |
| **25** | `HTGE_SUBCOVER` | `HTGE_SUBCOVER` | `ALPHADB` | **442** | **Yes** | Table to store policy-level subcovers and historical add-ons | `sat-inssurace-service`, `sat-motor` (FE) |
| **26** | `REVISION_CNGE_NOTE` | **`REV_CNGE_NOTE`** | `ALPHADB` | **631** | **Yes** | Table to store historical change revisions of basic vehicle details | `sat-inssurace-service`, `sat-motor` (FE) |
| **27** | `REVISION_CNGE_NOTE_MT` | **`REV_CNGE_NOTE_MT`** | `ALPHADB` | **640** | **Yes** | Table to store historical revisions of vehicle motor properties | `sat-inssurace-service`, `sat-motor` (FE) |
| **28** | `MKCL_PARTNER` | `MKCL_PARTNER` | `ALPHADB` | **582** | **Yes** | Table contains Customer Partner demographics | `sat-cp-service`, `sat-motor` (FE) |
| **29** | `MKCL_PARTNER_ADDRESS` | `MKCL_PARTNER_ADDRESS` | `ALPHADB` | **583** | **Yes** | Table to store mailing and billing addresses of Customer Partners | `sat-cp-service`, `sat-motor` (FE) |
| **30** | `MKCL_PARTNER_AGENT` | `MKCL_PARTNER_AGENT` | `ALPHADB` | **585** | **Yes** | Table to map Customer Partners to their designated Agents | `sat-cp-service`, `sat-motor` (FE) |
| **31** | `SASC_USER` | `SASC_USER` | `ALPHADB` | **723** | **Yes** | Table to store user profiles and portal credentials | `sat-bo-service`, `sat-motor` (FE) |
| **32** | `MKAG_AGENT` | `MKAG_AGENT` | `ALPHADB` | **558** | **Yes** | Table to store master broker and agent details (STATUS='A') | `sat-bo-service`, `sat-motor` (FE) |
| **33** | `MKAG_AGENT_DEALER` | `MKAG_AGENT_DEALER` | `ALPHADB` | **562** | **Yes** | Table to map agents to automotive dealer groups | `sat-bo-service`, `sat-motor` (FE) |
| **34** | `SAPM_CONFIG` | `SAPM_CONFIG` | `ALPHADB` | **671** | **Yes** | Table to store system-wide application configurations | `sat-bo-service`, `sat-inssurace-service`, `sat-motor` (FE) |
| **35** | `SAPM_PRODUCT_CONFIG` | `SAPM_PRODUCT_CONFIG` | `ALPHADB` | **680** | **Yes** | Table to store product and coverage setup rules (Product Configuration) | `sat-bo-service`, `sat-inssurace-service`, `sat-motor` (FE) |
| **36** | `SAPM_DOC_NUM_SEQGEN` | **`SAPM_DOCNUM_SEQGEN`** | `ALPHADB` | **675** | ❌ **No** | Table to store auto-increment sequences for documents (Draft, Proposal...) | `sat-inssurace-service` |
| **37** | `CMUW_NVIC` | `CMUW_NVIC` | `ALPHADB` | **128** | **Yes** | Table to store master NVIC (National Vehicle Identifier Code) codes | `sat-bo-service`, `sat-motor` (FE) |
| **38** | `CMUW_MODEL_VEH_INVIC` | **`CMUW_MODEL_VEH_NVIC`** | `ALPHADB` | **110** | **Yes** | Table maps NVIC codes to specific vehicle make, model, and variants | `sat-bo-service`, `sat-motor` (FE) |
| **39** | `CMGC_CODE_DET` | `CMGC_CODE_DET` | `ALPHADB` | **26** | **Yes** | Table to store master List-of-Values (LOV) codes / code details | `sat-bo-service`, `sat-motor` (FE) |
| **40** | `CMUW_MTVEHUSE` | `CMUW_MTVEHUSE` | `ALPHADB` | **122** | **Yes** | Table to store vehicle usage classifications (e.g. private, commercial) | `sat-bo-service`, `sat-motor` (FE) |
| **41** | `CMUW_MTVEHUSE_NCD` | `CMUW_MTVEHUSE_NCD` | `ALPHADB` | **126** | **Yes** | Table to store maximum NCD limits grouped by vehicle usage types | `sat-bo-service`, `sat-motor` (FE) |
| **42** | `CLAIM_DETAILS` | **`HTCL_CLAIM`** | `ALPHADB` | **389** | **Yes** | Table to store master historical accident and insurance claim logs | `sat-inssurace-service`, `sat-motor` (FE) |
| **43** | `HISTORICAL_CLAIM` | **`HTCL_CLAIM_DET`** | `ALPHADB` | **390** | **Yes** | Table to store detailed breakdowns of historical claims | `sat-inssurace-service`, `sat-motor` (FE) | |

---

> [!CRITICAL]
> ### ⚠️ CẢNH BÁO HỆ THỐNG QUAN TRỌNG:
> 1. **Bảng `SAPM_DOCNUM_SEQGEN` có trạng thái `Exists in PG? = No`**:
>    Bảng này quản lý dải số nhảy (Sequence Generator) tự động cho toàn bộ hệ thống báo giá và số Cover Note. Việc bảng này không tồn tại trên PostgreSQL đồng nghĩa với việc cổng ứng dụng khi chuyển đổi sang chạy trên PG sẽ bị lỗi khi sinh số đơn nếu không cấu hình giải pháp Sequence tương đương trong PostgreSQL (ví dụ: PG Sequences). Ghi chú: Dữ liệu của bạn cũng chỉ ra bảng **`SAPM_DOCNUM_SEQ`** (bảng Sequence gốc) có trạng thái `No` trên PG.
> 2. **Chênh lệch số lượng bản ghi (Record Count Difference):**
>    Các bảng danh mục như `MKAG_AGENT`, `CMGC_BRANCH`, và `CMGC_CODE_DET` được ghi nhận có sự chênh lệch số lượng bản ghi giữa database nguồn DB2 và database đích PostgreSQL. Cần thực hiện kiểm tra đồng bộ dữ liệu (Data Reconciliation) để tránh mất mát danh mục đại lý hoặc mã cấu hình LOV.
> 3. **Nhóm bảng cần Split (`CNGE_NOTE`, `CNGE_NOTE_EXT`, `CNGE_RISK`):**
>    Các bảng giao dịch lõi được đánh dấu **"Required to split"** và chỉ thực hiện di trú dữ liệu phần mềm (Yes, data only).
