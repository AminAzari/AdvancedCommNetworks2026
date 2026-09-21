
# ماژول 12: موجودیت مدیریت تحرک (MME)

## 1. چرا MME وجود دارد
<div dir="rtl">
MME **موجودیت مرکزی صفحه کنترل** در معماری LTE/EPC است. دلیل وجود آن:
</div>
- **تخلیه RAN از سیگنالینگ هسته**: در 2G/3G، RNC/BSC سیگنالینگ قابل توجهی از شبکه هسته را مدیریت می‌کرد. معماری مسطح LTE تمام پایان‌دهی سیگنالینگ NAS (لایه غیر-دسترسی) را به MME منتقل کرد و طراحی eNodeB (eNB) را ساده کرد.
- **متمرکزسازی هوش صفحه کنترل**: احراز هویت، مدیریت تحرک، برقراری حامل و صفحه‌بندی همه توسط MME هماهنگ می‌شوند.
- **جداکردن صفحه کنترل از صفحه کاربر**: MME به هیچ ترافیک صفحه کاربری دست نمی‌زند — صرفاً سیگنالینگ را مدیریت می‌کند، که مقیاس‌پذیری مستقل CP و UP را ممکن می‌سازد.

### واسط‌های کلیدی

| واسط | گره همتا | پروتکل | هدف |
|-----------|-----------|----------|---------|
| S1-MME | eNB | S1-AP (SCTP) | انتقال NAS، سیگنالینگ تحویل |
| S6a | HSS | Diameter | بردارهای احراز هویت، داده اشتراک |
| S11 | S-GW | GTPv2-C | مدیریت حامل/نشست |
| S10 | MME دیگر | GTPv2-C | تحویل بین-MME، انتقال زمینه |
| S3 | SGSN | GTPv2-C | تحرک 2G/3G ↔ LTE |

---

## 2. توابع MME

### 2.1 پایان‌دهی سیگنالینگ NAS
- MME نقطه انتهایی NAS است — تمام پیام‌های NAS از UE را پایان می‌دهد (که از طریق eNB با S1-AP به‌صورت شفاف منتقل می‌شوند).
- پیام‌های NAS بین UE و MME محافظت‌شده با یکپارچگی و رمزگذاری‌شده هستند.

### 2.2 مدیریت تحرک
- مدیریت ناحیه ردیابی (TA) و تخصیص فهرست TA
- سیگنالینگ تحویل (مبتنی بر S1 و X2 با دخالت MME)
- تحرک بین-RAT (به/از 2G/3G از طریق S3/S10)

### 2.3 احراز هویت و امنیت
- بردارهای احراز هویت را از HSS بازیابی می‌کند (از طریق S6a)
- EPS-AKA (احراز هویت و توافق کلید) را اجرا می‌کند
- کلیدهای NAS (K_NASint، K_NASenc) و کلیدهای AS (K_eNB) را مشتق می‌کند
- NAS Security Mode Command را آغاز می‌کند

### 2.4 مدیریت حامل
- حامل‌های پیش‌فرض EPS را در حین اتصال برقرار می‌کند
- فعال‌سازی/اصلاح/غیرفعال‌سازی حامل اختصاصی را هماهنگ می‌کند
- با S-GW (S11) و به‌طور غیرمستقیم با P-GW تعامل دارد

### 2.5 صفحه‌بندی
- UEها را در حالت ECM-IDLE هنگام رسیدن داده پیوند پایین‌سو صفحه‌بندی می‌کند
- پیام‌های S1-AP Paging را به تمام eNBها در فهرست TA ثبت‌شده UE می‌فرستد

### 2.6 دسترسی‌پذیری در حالت بیکار
- زمینه UE را حتی وقتی UE در IDLE (ECM-IDLE) است حفظ می‌کند
- می‌داند UE ممکن است در کدام نواحی ردیابی باشد
- هنگام رسیدن داده یا سیگنالینگ، صفحه‌بندی را فعال می‌کند

### 2.7 انتخاب، استخرسازی و توازن مجدد MME
- چندین MME به همان ناحیه خدمات می‌دهند (استخر MME)
- توازن بار از طریق ظرفیت نسبی (عامل وزن) که به eNBها اعلام می‌شود
- توازن مجدد: انتقال تدریجی زمینه‌های UE هنگام افزودن/حذف MMEها

---

## 3. پشته پروتکل NAS

NAS (لایه غیر-دسترسی) بین UE و MME عمل می‌کند و نسبت به eNB شفاف است.

### 3.1 EMM — مدیریت تحرک EPS

ثبت‌نام و تحرک UE را مدیریت می‌کند:
- Attach / Detach
- به‌روزرسانی ناحیه ردیابی (TAU)
- احراز هویت (EPS-AKA)
- کنترل حالت امنیتی
- تخصیص مجدد GUTI
- درخواست خدمات

**حالت‌های EMM:**
- EMM-DEREGISTERED: UE ثبت‌نام نشده است
- EMM-REGISTERED: UE به شبکه متصل است

### 3.2 ESM — مدیریت نشست EPS

اتصالات PDN و حامل‌های EPS را مدیریت می‌کند:
- درخواست/پذیرش اتصال PDN
- تخصیص منابع حامل
- اصلاح / غیرفعال‌سازی حامل
- قطع اتصال PDN

**ESM در حین اتصال به‌صورت حمل‌شده روی EMM عمل می‌کند** (پیام ESM درون Attach Request جای می‌گیرد).

---

## 4. رویه‌های تفصیلی

### 4.1 اتصال اولیه LTE (جریان کامل)

```mermaid
sequenceDiagram
    participant UE
    participant eNB
    participant MME
    participant HSS
    participant SGW as S-GW
    participant PGW as P-GW

    Note over UE,PGW: Initial Attach Procedure (~15 steps)
    
    UE->>eNB: 1. RRC Connection Request
    eNB->>UE: 2. RRC Connection Setup
    UE->>eNB: 3. RRC Connection Setup Complete<br/>(contains NAS: Attach Request + PDN Connectivity Request)
    eNB->>MME: 4. S1-AP: Initial UE Message<br/>(NAS: Attach Request + ESM: PDN Connectivity Req)
    
    Note over MME,HSS: Authentication Phase
    MME->>HSS: 5. S6a: Authentication-Information-Request (AIR)
    HSS->>MME: 6. S6a: Authentication-Information-Answer (AIA)<br/>(RAND, AUTN, XRES, K_ASME)
    MME->>UE: 7. NAS: Authentication Request (RAND, AUTN)
    UE->>MME: 8. NAS: Authentication Response (RES)
    
    Note over MME,UE: Security Activation
    MME->>UE: 9. NAS: Security Mode Command<br/>(selected algorithms, NAS-MAC)
    UE->>MME: 10. NAS: Security Mode Complete (ciphered+integrity)
    
    Note over MME,HSS: Location Update
    MME->>HSS: 11. S6a: Update-Location-Request (ULR)
    HSS->>MME: 12. S6a: Update-Location-Answer (ULA)<br/>(subscription data, APN config)
    
    Note over MME,PGW: Session/Bearer Establishment
    MME->>SGW: 13. S11: Create Session Request<br/>(IMSI, bearer QoS, P-GW address)
    SGW->>PGW: 14. S5: Create Session Request
    PGW->>SGW: 15. S5: Create Session Response<br/>(IP addr allocated, bearer context)
    SGW->>MME: 16. S11: Create Session Response<br/>(S-GW F-TEID, bearer context)
    
    Note over MME,UE: Attach Accept
    MME->>eNB: 17. S1-AP: Initial Context Setup Request<br/>(NAS: Attach Accept + Activate Default Bearer Context,<br/>security context, UE capabilities)
    eNB->>UE: 18. RRC Reconfiguration (DRB setup)<br/>+ NAS: Attach Accept
    UE->>eNB: 19. RRC Reconfiguration Complete
    eNB->>MME: 20. S1-AP: Initial Context Setup Response<br/>(eNB F-TEID for S1-U)
    UE->>MME: 21. NAS: Attach Complete<br/>(+ ESM: Activate Default EPS Bearer Context Accept)
    MME->>SGW: 22. S11: Modify Bearer Request (eNB address/TEID)
    SGW->>MME: 23. S11: Modify Bearer Response
```

**نکات کلیدی:**
- گام‌های 7-10: EPS-AKA + فعال‌سازی امنیت NAS
- گام‌های 13-16: برقراری نشست GTP-C در سراسر S11/S5
- درخواست اتصال، یک PDN Connectivity Request حمل‌شده (ESM) را حمل می‌کند
- UE آدرس IP خود را از P-GW می‌گیرد (گام 15)


### 4.2 احراز هویت و حالت امنیتی

```mermaid
sequenceDiagram
    participant UE
    participant MME
    participant HSS

    Note over UE,HSS: EPS-AKA Authentication
    MME->>HSS: Authentication-Information-Request<br/>(IMSI, SN-ID, visited PLMN)
    HSS->>MME: Authentication-Information-Answer<br/>(EPS-AV: RAND, AUTN, XRES, K_ASME)
    
    MME->>UE: Authentication Request (RAND, AUTN)
    Note over UE: UE verifies AUTN (mutual auth)<br/>Computes RES, K_ASME
    UE->>MME: Authentication Response (RES)
    Note over MME: Verifies RES == XRES
    
    Note over UE,MME: NAS Security Mode
    Note over MME: Derives K_NASint, K_NASenc<br/>Selects EIA/EEA algorithms
    MME->>UE: Security Mode Command<br/>(replayed UE security capabilities,<br/>selected algorithms, NAS-MAC)
    Note over UE: Verifies NAS-MAC<br/>Derives NAS keys
    UE->>MME: Security Mode Complete<br/>(integrity + ciphered from here on)
    
    Note over UE,MME: All subsequent NAS messages<br/>are integrity-protected and ciphered
```

**سلسله‌مراتب مشتق‌سازی کلید:**
```
K (USIM) → CK, IK → K_ASME → K_NASint, K_NASenc, K_eNB → K_UPenc, K_RRCint, K_RRCenc
```

### 4.3 به‌روزرسانی ناحیه ردیابی (TAU)

**محرک‌ها:**
- **TAU دوره‌ای**: تایمر T3412 منقضی می‌شود (پیش‌فرض حدود 54 دقیقه) — تأیید می‌کند UE هنوز قابل دسترسی است
- **TAU با محرک تحرک**: UE به سلولی می‌رود که TAC آن در فهرست TA آن نیست

```mermaid
sequenceDiagram
    participant UE
    participant eNB
    participant NewMME as New MME
    participant OldMME as Old MME
    participant HSS
    participant SGW as S-GW

    UE->>eNB: RRC: Service Request / TAU trigger
    eNB->>NewMME: S1-AP: Initial UE Message<br/>(NAS: TAU Request, old GUTI)
    
    alt Inter-MME TAU (UE moved to new MME pool area)
        NewMME->>OldMME: S10: Context Request (old GUTI)
        OldMME->>NewMME: S10: Context Response<br/>(MM context, bearer contexts)
        NewMME->>HSS: S6a: Update Location Request
        HSS->>NewMME: S6a: Update Location Answer
        HSS->>OldMME: S6a: Cancel Location (old MME deregistered)
        NewMME->>SGW: S11: Modify Bearer Request (new MME TEID)
        SGW->>NewMME: S11: Modify Bearer Response
    else Intra-MME TAU (same MME, new TA)
        Note over NewMME: Update TA List locally
    end
    
    NewMME->>UE: NAS: TAU Accept<br/>(new GUTI, new TA List, T3412 value)
    UE->>NewMME: NAS: TAU Complete
```

### 4.4 رویه جداسازی

#### جداسازی با شروع UE

```mermaid
sequenceDiagram
    participant UE
    participant eNB
    participant MME
    participant HSS
    participant SGW as S-GW
    participant PGW as P-GW

    UE->>MME: NAS: Detach Request<br/>(Detach Type: EPS detach / IMSI detach / combined)
    MME->>SGW: S11: Delete Session Request
    SGW->>PGW: S5: Delete Session Request
    PGW->>SGW: S5: Delete Session Response
    SGW->>MME: S11: Delete Session Response
    MME->>HSS: S6a: Purge-UE-Request (if switch-off)
    HSS->>MME: S6a: Purge-UE-Answer
    MME->>UE: NAS: Detach Accept
    MME->>eNB: S1-AP: UE Context Release Command
    eNB->>UE: RRC Connection Release
    eNB->>MME: S1-AP: UE Context Release Complete
```

#### جداسازی با شروع شبکه

```mermaid
sequenceDiagram
    participant MME
    participant UE
    participant eNB
    participant SGW as S-GW
    participant PGW as P-GW

    Note over MME: Trigger: subscription withdrawal,<br/>O&M intervention, MME overload
    MME->>UE: NAS: Detach Request<br/>(re-attach required / re-attach not required)
    UE->>MME: NAS: Detach Accept
    MME->>SGW: S11: Delete Session Request
    SGW->>PGW: S5: Delete Session Request
    PGW->>SGW: S5: Delete Session Response
    SGW->>MME: S11: Delete Session Response
    MME->>eNB: S1-AP: UE Context Release Command
    eNB->>MME: S1-AP: UE Context Release Complete
```


### 4.5 صفحه‌بندی (رویه صفحه‌بندی S1)

```mermaid
sequenceDiagram
    participant Internet
    participant PGW as P-GW
    participant SGW as S-GW
    participant MME
    participant eNB1 as eNB (TA1)
    participant eNB2 as eNB (TA2)
    participant UE

    Note over UE: UE is in ECM-IDLE (RRC released)
    Internet->>PGW: Downlink IP packet for UE
    PGW->>SGW: GTP-U: DL data (S5 tunnel)
    Note over SGW: No S1-U tunnel (UE idle)<br/>Buffer DL packet
    SGW->>MME: S11: Downlink Data Notification
    MME->>SGW: S11: Downlink Data Notification Ack
    
    Note over MME: Page UE in all TAs in TA List
    par Paging in TA1
        MME->>eNB1: S1-AP: Paging<br/>(UE identity: S-TMSI, TAI list, DRX)
    and Paging in TA2
        MME->>eNB2: S1-AP: Paging<br/>(UE identity: S-TMSI, TAI list, DRX)
    end
    
    Note over eNB1,eNB2: eNBs transmit paging on PCH<br/>in UE's paging occasion (DRX cycle)
    eNB1->>UE: RRC: Paging (s-TMSI)
    UE->>eNB1: RRC Connection Request (Service Request trigger)
    Note over UE,MME: Service Request procedure follows
```

### 4.6 درخواست خدمات (UE از IDLE ← CONNECTED)

```mermaid
sequenceDiagram
    participant UE
    participant eNB
    participant MME
    participant SGW as S-GW

    Note over UE: UE in ECM-IDLE has UL data<br/>or was paged
    UE->>eNB: RRC Connection Request
    eNB->>UE: RRC Connection Setup
    UE->>eNB: RRC Connection Setup Complete<br/>(NAS: Service Request, short-MAC-I)
    eNB->>MME: S1-AP: Initial UE Message<br/>(NAS: Service Request)
    
    Note over MME: Verify short-MAC-I<br/>(integrity check with K_NASint)
    
    MME->>eNB: S1-AP: Initial Context Setup Request<br/>(E-RAB setup list, security context, K_eNB)
    eNB->>UE: RRC Connection Reconfiguration<br/>(DRB re-establishment)
    UE->>eNB: RRC Connection Reconfiguration Complete
    eNB->>MME: S1-AP: Initial Context Setup Response<br/>(eNB S1-U TEID/address)
    
    MME->>SGW: S11: Modify Bearer Request<br/>(eNB address, S1-U TEID per bearer)
    SGW->>MME: S11: Modify Bearer Response
    Note over SGW: Flush buffered DL packets<br/>via new S1-U tunnel
    SGW->>eNB: GTP-U: Buffered DL data
    eNB->>UE: DL data on DRB
```

---

## 5. مدیریت حامل

### 5.1 برقراری حامل پیش‌فرض
- به‌طور خودکار در حین اتصال اولیه ایجاد می‌شود (بخشی از Create Session)
- یک حامل پیش‌فرض در هر اتصال PDN (همیشه فعال، غیر-GBR)
- QCI معمولاً 9 (اینترنت بهترین تلاش) یا 5 (سیگنالینگ IMS)
- بدون قطع اتصال PDN نمی‌تواند غیرفعال شود

### 5.2 فعال‌سازی حامل اختصاصی

```mermaid
sequenceDiagram
    participant PCRF
    participant PGW as P-GW
    participant SGW as S-GW
    participant MME
    participant eNB
    participant UE

    PCRF->>PGW: Gx: RAR (PCC rules, new SDF)
    PGW->>SGW: S5: Create Bearer Request<br/>(TFT, QoS: QCI, GBR/MBR)
    SGW->>MME: S11: Create Bearer Request
    MME->>eNB: S1-AP: E-RAB Setup Request<br/>(NAS: Activate Dedicated EPS Bearer Context Request)
    eNB->>UE: RRC Reconfiguration (new DRB)
    UE->>eNB: RRC Reconfiguration Complete
    eNB->>MME: S1-AP: E-RAB Setup Response
    UE->>MME: NAS: Activate Dedicated EPS Bearer Context Accept
    MME->>SGW: S11: Create Bearer Response
    SGW->>PGW: S5: Create Bearer Response
```

### 5.3 اصلاح حامل اختصاصی
- با محرک تغییر سیاست PCRF (مثلاً ارتقای QoS در حین تماس VoLTE)
- از Update Bearer Request/Response در سراسر S5 ← S11 ← S1-AP استفاده می‌کند

### 5.4 غیرفعال‌سازی حامل اختصاصی
- با محرک PCRF (حذف سیاست)، P-GW یا MME
- از Delete Bearer Request/Response استفاده می‌کند
- غیرفعال‌سازی حامل پیش‌فرض = قطع اتصال PDN (تمام حامل‌های آن PDN حذف می‌شوند)

---

## 6. انتخاب MME

### 6.1 انتخاب مبتنی بر DNS
- eNB نام FQDN مربوط به MME را از TAI حل می‌کند ← DNS فهرست IPهای MME را بازمی‌گرداند
- قالب: `tac-XXXX.tac-YYYY.mme.epc.mnc<MNC>.mcc<MCC>.3gppnetwork.org`

### 6.2 توازن بار
- هر MME یک **ظرفیت نسبی MME** (وزن، 0-255) را از طریق S1 Setup اعلام می‌کند
- eNB UEهای جدید را به‌تناسب در بین اعضای استخر توزیع می‌کند
- مقدار ظرفیت بالاتر = UEهای بیشتر به آن MME هدایت می‌شوند

### 6.3 GUMMEI — شناسه یکتای جهانی MME
```
GUMMEI = MCC + MNC + MME Group ID (MMEGI) + MME Code (MMEC)
```
- MMEGI استخر MME را شناسایی می‌کند
- MMEC یک MME مشخص درون استخر را شناسایی می‌کند
- GUTI = GUMMEI + M-TMSI (هویت موقت UE)

---

## 7. S1-Flex و استخرسازی MME

### مفهوم
- **S1-Flex**: هر eNB به **چندین MME** در یک استخر متصل می‌شود (نه فقط یکی)
- **ناحیه استخر MME**: ناحیه جغرافیایی که توسط مجموعه‌ای از MMEها خدمات می‌گیرد
- مزایا:
  - **افزونگی**: اگر یک MME شکست بخورد، بقیه به UEها خدمات می‌دهند
  - **اشتراک بار**: UEهای جدید بر اساس وزن توزیع می‌شوند
  - **افزونگی جغرافیایی**: اعضای استخر می‌توانند در مراکز داده مختلف باشند
  - **مقیاس‌دهی تدریجی**: افزودن/حذف MMEها بدون وقفه سرویس

### اضافه‌بار و توازن مجدد MME
- MME یک **Overload Start** به eNBها می‌فرستد ← eNB اتصالات جدید را به سایر اعضای استخر هدایت می‌کند
- **توازن مجدد**: MME از UEها می‌خواهد دوباره متصل شوند تا بار را بازتوزیع کند (از طریق جداسازی ضمنی با نشانه اتصال مجدد)

---

## 8. تحول: MME ← AMF (5G)

| جنبه | MME در LTE | AMF در 5G |
|--------|---------|--------|
| معماری | NE یکپارچه | میکروسرویس (مبتنی بر SBI) |
| پروتکل | GTPv2-C، Diameter | HTTP/2، JSON (SBI)، NGAP |
| NAS | EMM + ESM (ترکیبی) | 5G-NAS (فقط MM؛ SM به SMF جدا شد) |
| مدیریت نشست | MME سیگنالینگ حامل را انجام می‌دهد | کاملاً به SMF واگذار شده |
| شبکه‌بندی | N/A | آگاه به برش، مسیریابی به SMF صحیح در هر برش |
| ثبت‌نام | Attach/TAU | Registration (یکپارچه) |
| حالت | حالت‌دار | طراحی‌شده برای بدون-حالت (UDSF) |
| واسط با RAN | S1-AP | NGAP (N2) |
| واسط با UDM | Diameter S6a | Nudm (SBI) |

**تغییرات کلیدی:**
1. **جداسازی مدیریت نشست**: MME هم تحرک و هم حامل‌ها را انجام می‌داد؛ در 5G، AMF فقط تحرک/امنیت را انجام می‌دهد، SMF نشست‌ها را مدیریت می‌کند
2. **واسط خدمات‌محور**: AMF خدمات Namf را ارائه می‌دهد که توسط سایر NFها از طریق HTTP/2 مصرف می‌شود
3. **شبکه‌بندی**: AMF انتخاب برش (تعامل NSSF) را انجام می‌دهد و به SMF مختص برش مسیریابی می‌کند
4. **طراحی بدون حالت**: زمینه UE می‌تواند به‌صورت خارجی (UDSF) برای تاب‌آوری ذخیره شود
5. **ثبت‌نام یکپارچه**: بدون Attach در برابر TAU جداگانه — رویه Registration واحد هر دو را مدیریت می‌کند

