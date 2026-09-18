---
name: coder
description: Triển khai bản kế hoạch nằm ở PromtAI/.codex/results/plan.md. Chặng thứ hai của dây chuyền, chạy ngay sau planner.
tools: Read, Write, Edit, Grep, Glob, Bash
model: luna max
---
Bạn là chuyên gia triển khai.
1. Đọc trọn file `PromtAI/.codex/results/plan.md`. Nếu trong đó có mục **CÂU HỎI CÒN BỎ NGỎ**, hãy DỪNG LẠI và nêu các câu hỏi đó ra, đừng tự đoán.
2. Xây đúng những gì bản kế hoạch mô tả. Bám theo các quy ước mà nó chỉ định. Không thêm tính năng nào mà kế hoạch không yêu cầu.
3. Ghi tóm tắt ngắn ra `PromtAI/.codex/results/change.md`, gồm: Những file đã thay đổi, mỗi chỗ sửa để làm gì, và chỗ nào Tester nên soi kỹ.

Code bạn viết phải khớp phong cách sẵn có của repo. Không dọn dẹp, không cải tiến những đoạn code không liên quan, không làm gì nằm ngoài phạm vi bản kế hoạch.



<!-- CODEGRAPH_START -->
## CodeGraph

In repositories indexed by CodeGraph (a `.codegraph/` directory exists at the repo root), reach for it BEFORE grep/find or reading files when you need to understand or locate code:

- **MCP tools** (when available): `codegraph_explore` answers most code questions in one call — the relevant symbols' verbatim source plus the call paths between them. `codegraph_node` returns one symbol's source + callers, or reads a whole file with line numbers. If the tools are listed but deferred, load them by name via tool search.
- **Shell** (always works): `codegraph explore "<symbol names or question>"` and `codegraph node <symbol-or-file>` print the same output.

If there is no `.codegraph/` directory, skip CodeGraph entirely — indexing is the user's decision.
<!-- CODEGRAPH_END -->

# PHẦN A --- BACKEND (.NET / Clean Architecture + CQRS + MediatR)

**MUST** 
    - `Project.Domain` không reference bất kỳ project nào khác. Chỉ Entity, ValueObject, Specification, interface thuần. 
    - `Project.Application` chứa toàn bộ business logic (Command/Query/Handler/Service/Dto). 
    - `Project.AzureFunctions` chỉ là lớp vỏ HTTP: auth → deserialize → `_mediator.Send` → trả response.
**Không có business logic trong Functions.** 
- Namespace = đường dẫn thư mục, **trừ** thư mục có tiền tố `_`: thư mục `_Common/Base` → namespace
`Project.Application.Common.Base` (dấu `_` chỉ để đẩy folder lên đầu Solution Explorer, không đưa vào namespace). - 1 file = 1 type, tên file
= tên type.

**AVOID** - `Project.Application._Common.Base` --- có một file đang sai như vậy (`SearchBaseDto.cs`), không lặp lại.

## A2. Tổ chức thư mục theo feature

    Project.Application/<Feature>/
    ├── Commands/<ActionName>/          # 3 file cùng tên gốc
    │   ├── <Action><Feature>Command.cs
    │   ├── <Action><Feature>CommandHandler.cs
    │   └── <Action><Feature>CommandResult.cs
    ├── Queries/<ActionName>/
    │   ├── <Action><Feature>Query.cs
    │   ├── <Action><Feature>QueryHandler.cs
    │   └── <Action><Feature>QueryResult.cs
    ├── Dtos/                           # DTO dùng chung trong feature
    ├── Services/                       # I<X>Service.cs + <X>Service.cs
    └── Helpers/                        # static helper riêng feature

**MUST** 
- Mỗi Command/Query nằm trong **thư mục riêng**, đủ 3 file (Command / Handler / Result). Không gộp 3 class vào 1 file. - Tên thư
mục = tên action (`Create`, `Update`, `Delete`, `Search`, `GetById`, `Export`, `GetOrganizationChart`...). - Service dùng chung nhiều feature
→ `_Common/`. Service chỉ 1 feature dùng → `<Feature>/Services/`.

## A3. Quy tắc đặt tên

  -----------------------------------------------------------------------------------
  Đối tượng               Quy tắc                   Ví dụ trong code
  ----------------------- ------------------------- ---------------------------------
  Class / Method /        `PascalCase`              `EquipmentProfile`,
  Property                                          `GetCurrentScopeAsync`

  Interface               `I` + PascalCase          `IEquipmentAccessScopeService`

  Biến local / tham số    `camelCase`               `cancellationToken`,
                                                    `allowedTestGroupIds`

  Field private           `_camelCase`              `_context`, `_mediator`,
                                                    `_scopeService`

  Const                   `PascalCase`              `DefaultEquipmentCodePattern`

  Enum + member           `PascalCase`, số nguyên   `EquipmentStatus.Drafted = 0`
                          rõ ràng                   

  Entity                  Danh từ số ít             `EquipmentProfile`,
                                                    `Procurement`, `Attachment`

  DbSet                   Danh từ số nhiều          `EquipmentProfiles`,
                                                    `Procurements`

  DTO                     `<Tên>Dto`                `EquipmentProfileDto`,
                                                    `AttachmentInputDto`

  DTO tìm kiếm            `<Tên>SearchDto`          `EquipmentProfileSearchDto`

  DTO export              `<Tên>ExportDto`          `EquipmentProfileExportDto`

  Service                 `<Tên>Service` +          `MailService` / `IMailService`
                          `I<Tên>Service`           

  EF Configuration        `<Entity>Configuration`   `EquipmentProfileConfiguration`

  Azure Function class    `<Feature>Functions`      `EquipmentFunctions`

  Permission const        `<mod>.<res>.<action>`    `"eqp.profile.request-update"`
                          chữ thường, gạch nối      
  -----------------------------------------------------------------------------------

**MUST** 
    - Method `async` kết thúc bằng `Async`: `GetCurrentScopeAsync`, `SaveChangesAsync`. (Ngoại lệ duy nhất: `Handle` của MediatR --- do
interface quy định.) - Không viết tắt tuỳ tiện. Chỉ dùng viết tắt đã là từ vựng của hệ thống: `Eq` (Equipment), `Cmp`, `Org`, `Dto`, `Id`. Không
đặt `usr`, `eqp1`, `tmp2`. - Boolean bắt đầu bằng `Is/Has/Requires/Can`:
`IsActive`, `IsSucceed`, `HasFlag`, `RequiresCalibration`,
`CanAccessEquipmentAsync`. - Method trả về `Task` mà chỉ ném exception khi fail → prefix `Ensure`: `EnsureCanAccessEquipmentAsync`. Trả `bool`
→ prefix `Can`/`Is`.

## A4. Command / Query (CQRS)

**MUST** 
- **Command** = ghi dữ liệu. 
- **Query** = chỉ đọc, không
`SaveChangesAsync`. - Command là `class` (có nhiều property, cần gán
thêm ở Function layer):


## A5. Handler

**MUST** 
- Dùng **primary constructor** + gán vào field `readonly`:


Chỉ dùng exception cho lỗi hệ thống / vi phạm quyền
(`ForbiddenAccessException`, `NotFoundException` trong
`_Common/Exceptions`). - **Guard trước, happy-path sau.** Validate →
load & kiểm tra tồn tại → kiểm tra quyền scope → mutate →
`SaveChangesAsync` → return. Không lồng `if` nhiều tầng. - Chỉ gọi
`SaveChangesAsync` **một lần**, ở cuối handler, để cả nghiệp vụ nằm
trong 1 transaction ngầm của EF. - Handler dài **SHOULD** tách logic phụ
thành `private static` method trong chính handler: `Validate(...)`,
`ApplyProfile(...)`, `NormalizeAttachments(...)`. Đặt `static` nếu không
dùng field instance. - Handler **không** được gọi trực tiếp handler
khác. Muốn tái sử dụng nghiệp vụ: gọi qua `_mediator.Send(...)` (xem
`CreateOrganizationUnitCommandHandler`) hoặc tách ra Service.

**AVOID** 
- Handler \> \~300 dòng mà không tách private method. 
- Ghi `DateTime.Now` rải rác: khai báo `var now = DateTime.UtcNow;` một lần
đầu handler rồi dùng lại, để mọi bản ghi cùng nghiệp vụ có chung
timestamp.

## A6. Truy vấn EF Core trong handler

**MUST** 
- Query chỉ để đọc → **luôn** `.AsNoTracking()`. - **Luôn** filter đủ 3 điều kiện chuẩn của hệ thống:

``` csharp
x.TenantId == command.TenantId
&& x.ActiveFlag == (byte)ActiveFlag.Active
&& x.IsActive
```

Thiếu `TenantId` = lỗi bảo mật multi-tenant, reject PR ngay. - Soft
delete: set `ActiveFlag = (byte)ActiveFlag.Inactive`, **không**
`Remove()`. - Search text dùng
`EF.Functions.Like((x.Field ?? string.Empty).ToLower(), "%" + keyword + "%")`
với `keyword` đã `Trim().ToLower()` sẵn. - Lọc theo khoảng ngày:
`>= from.Date` và `< to.Date.AddDays(1)` (exclusive), không dùng
`<= to`. - Phân trang theo `TableRequest` (`Draw`/`Start`/`Length`), trả
`DataTableResponse<T>` với `Data`, `Draw`, `RecordsFiltered`,
`RecordsTotal`. - Tránh N+1: load list rồi build `Dictionary` bằng
`ToDictionaryAsync` / `GroupBy().ToDictionary()`, sau đó `TryGetValue`
khi map --- như `SearchEquipmentProfilesQueryHandler`.

## A7. Entity & EF Configuration

**MUST** 
- Entity kế thừa base có sẵn (`MasterEntity`) --- không tự khai
lại `Id`, `CreateBy`, `CreateDate`, `TenantId`, `ActiveFlag`. - Mapping
đặt bảng/schema bằng attribute trên entity:


**AVOID** - Hard-code connection string trong `OnConfiguring` (hiện
`ApplicationContext.cs` đang có --- **không** copy pattern này, và
**không** commit thêm credential nào).

## A8. Service dùng chung

**MUST** - Mỗi service có cặp `I<X>Service` + `<X>Service`, cùng thư mục
`Services/`. - Interface chỉ khai method `Async` trả `Task`/`Task<T>`,
có `CancellationToken` ở tham số cuối. - Đăng ký DI tập trung tại
`Project.Application/ConfigureServices.cs` (`AddApplicationServices`).
Mặc định `AddScoped`. `AddSingleton` chỉ cho stateless/config
(`ITokenValidationService`). - Service được inject vào handler qua
constructor --- **không** `new` service trong handler, không dùng
service locator. - Service có cache trong 1 request → cache bằng private
field (`_cachedScope`) và vì scope = per-request nên an toàn. Không dùng
`static` field để cache dữ liệu theo user/tenant. - Logic dùng \> 1
feature → đưa vào `_Common/` (Mail, Excel, Attachment,
DocumentProcessing, ObjectComparer, HashToken). Đừng copy-paste giữa các
feature.

## A9. Azure Functions layer

**MUST** - Class kế thừa `BaseFunctions`, dùng primary constructor,
logger riêng:

``` csharp
private readonly ILogger _logger = loggerFactory.CreateLogger<EquipmentFunctions>();
```

-   Mẫu chuẩn cho mọi endpoint:

``` csharp
[Function("CreateEquipment")]
public async Task<HttpResponseData> CreateEquipment(
[HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = "{tenantId}/equipment")] HttpRequestData req,
string tenantId,
CancellationToken cancellationToken)
{
var (isValidToken, userEmail) = await IsValidTokenAsync(req);
if (!isValidToken) return req.CreateResponse(HttpStatusCode.Unauthorized);
currentUser.SetUser(userEmail, tenantId);

try
{
// deserialize → gán TenantId/CreateBy/UpdateBy → _mediator.Send
return req.CreateJsonResponse(HttpStatusCode.OK, result);
}
catch (ForbiddenAccessException ex)
{
return req.CreateJsonResponse(HttpStatusCode.Forbidden, ex.Message);
}
catch (Exception ex)
{
_logger.LogError(ex.ToString());
return req.CreateJsonResponse(HttpStatusCode.InternalServerError, ex.InnerException?.Message ?? ex.Message);
}
}
```

-   Tên `[Function("...")]` = tên method, `PascalCase`, unique toàn app.
-   Route: `{tenantId}/<resource>/<sub-resource>`, **kebab-case**, danh
    từ số nhiều (`{tenantId}/equipment/categories/eq-types`). Không đặt
    động từ trong route (dùng HTTP verb).
-   Luôn kiểm tra token **trước**, rồi
    `currentUser.SetUser(userEmail, tenantId)`.
-   Function **MUST NOT** truy cập `ApplicationContext` trực tiếp.

------------------------------------------------------------------------

# PHẦN B --- FRONTEND (React 18 + TS + Fluent UI v9 + RxJS + Module Federation)

## B1. Cấu trúc thư mục

    src/
    ├── constants/       # enum, hằng số  → app-const.ts, employee-const.ts...
    ├── context-…        # trạng thái global (BehaviorSubject)
    ├── custom-hooks/    # useBehaviorSubject, useFetch, usePrevious...
    ├── hooks/
    │   ├── blocks/      # page-level: 1 thư mục / 1 module nghiệp vụ
    │   └── components/  # component tái sử dụng toàn app
    ├── i18n/{en,vi}/HR.json
    ├── layouts/
    ├── models/<domain>-models/   # interface TS + hàm Init
    ├── services/        # <domain>-service.ts — mọi lời gọi API
    └── utilities/       # routers.ts, utilities.ts

**MUST** - Component dùng ở ≥ 2 module → `hooks/components/`. Chỉ 1
module dùng → nằm trong thư mục module đó. - Không gọi API trực tiếp
trong component. **Mọi** request đi qua `services/<domain>-service.ts`.

## B2. Đặt tên

  -------------------------------------------------------------------------
  Đối tượng               Quy tắc                 Ví dụ
  ----------------------- ----------------------- -------------------------
  File component          `PascalCase.tsx`        `ListBusinessUnit.tsx`,
                                                  `CountryDirectory.tsx`

  File service / model /  `kebab-case.ts`         `country-service.ts`,
  util                                            `business-unit.ts`

  Thư mục model           `<domain>-models`       `category-models`,
                                                  `compliance-models`

  Component / Interface / `PascalCase`            `FlagCard`,
  Type                                            `CountryModel`

  Model interface         `<Tên>Model`            `CountryModel`,
                                                  `EmployeeInfoModel`

  Hàm khởi tạo model      `Init<Tên>`             `InitCountry`,
                                                  `InitBaseModel`

  Service object          `<Tên>Service`          `CountryService`,
                                                  `CategoryService`

  BehaviorSubject         `camelCase` + hậu tố    `currentCountry$`,
                          `$`                     `userPermission$`

  Subject cho bản ghi     `detail<Tên>$`          `detailBusinessUnit$`,
  đang xem/sửa                                    `detailRole$`

  Biến / hàm              `camelCase`             `handleSubmit`,
                                                  `recordsTotal`

  Handler sự kiện         `handle<X>` / prop      `handleEditClick`,
                          `on<X>`                 `onSettingClick`

  Custom hook             `use<X>`                `useBehaviorSubject`,
                                                  `useFetch`

  Hằng số                 `UPPER_SNAKE` hoặc      `FLAGS_BASE`,
                          `PascalCase`            `EmptyGuid`,
                                                  `DefaultPageSize`

  Enum + member           `PascalCase`            `BaseStatus.Actived`
  -------------------------------------------------------------------------

**MUST** - **Property của model dùng `PascalCase`** (`Id`, `Name`,
`IsActive`, `CountryCode`) vì map 1-1 với DTO C#. State/biến local thì
`camelCase`. Ngoại lệ đã tồn tại: `PaginatedData<T>` dùng
`data`/`recordsTotal` (khớp response DataTable của backend) --- giữ
nguyên, đừng đổi. - Bộ file CRUD của 1 danh mục **MUST** đủ và đúng tên:
`Main<X>.tsx` (điều hướng create/edit theo `id` trên query),
`List<X>.tsx`, `Create<X>.tsx`, `Edit<X>.tsx`, `Delete<X>.tsx`,
`Form<X>.tsx` (form dùng chung cho Create & Edit).

## B3. Model

**MUST** - Model kế thừa base có sẵn: `BaseModel` → `CategoryEntity`
(`src/models/base-models/base.ts`). Không khai lại `Id`, `Name`,
`IsActive`, `CreateBy`, `UpdateDateDisplay`... - Mỗi model có hàm
`Init<Tên>()` trả object mặc định, spread từ `InitBaseModel`:

``` ts
export const InitCountry = (): CountryModel => ({
...InitBaseModel, Index: 0, Logo: "", URL: "", Managers: [], RegionId: "", CountryCode: "",
});
```

-   Field optional dùng `?`, không dùng `| undefined` thủ công.

## B4. Service (RxJS)

**MUST** - Service là **object literal** export named, không phải class:

``` ts
const CountryService = { getAllCountries(): Observable<CountryModel[]> { ... } };
export { CountryService };
```

-   Mọi method trả `Observable<T>`, gọi qua
    `fetchWithTenantIDAndErrorHandler({ suffix, headers, method, body })`.
-   URL dựng theo mẫu:
    `` `${suffixAPI}/api/${OnePortalTenantIdKeyToReplace}/<resource>/<action>` ``
    --- luôn có `OnePortalTenantIdKeyToReplace`, resource **kebab-case
    số nhiều**.
-   `.pipe(map((result: any) => result.response ?? <giá trị mặc định>))`
    --- luôn có fallback (`[]`, `false`, `null`), không để component
    nhận `undefined`.
-   Tên method service: `getAll<X>`, `get<X>ById`, `search<X>s`,
    `create<X>`, `update<X>`, `delete<X>`.
-   Cache dữ liệu tĩnh (country, category) bằng biến module-scope +
    `of(cached)` như `CountryService.getAllCountries`. Chỉ áp dụng cho
    dữ liệu ít đổi.
-   Import từ alias Module Federation, **không** import trực tiếp
    package: `libapp/react`, `libapp/rxjs`, `libapp/fluentv9`,
    `libapp/react-i18next`, `oneportal/services/...`,
    `oneportalutilities/router/...`.

## B5. Component

**MUST** - Function component + `export default` ở cuối file. Props khai
inline có kiểu:

``` ts
const CreateBusinessUnit = (props: { handleClose: () => void; handleSubmit: () => void }) => {
```

-   Subscribe trong component **MUST** có `.pipe(take(1))` và xử lý cả
    `next` + `error` (trong `error` nhớ `setLoading(false)`):

``` ts
CategoryService.searchBusinessUnits(model)
.pipe(take(1))
.subscribe({
next: (res) => { setData(res.data); setRecordsTotal(res.recordsTotal); setLoading(false); },
error: () => setLoading(false),
});
```

Stream sống lâu (BehaviorSubject) → dùng `useBehaviorSubject(subject$)`,
hook này tự `unsubscribe`. - Thông báo cho user **MUST** dùng
`NotifyService.pushNotify` với `intent: "success" | "error"`, nội dung
lấy từ `t("General.Notify.*")`. Không dùng `alert`, không `console.log`
trong code merge. - Mọi state loading của form/list:
`const [loading, setLoading] = React.useState(false);` + chặn
double-submit `if (loading) return;` ở đầu `handleSubmit`. - Điều hướng
dùng `useLink(appRouter, router, routers.<name>.path)`. Truyền tham số
theo mẫu `` `${routers.x.path}&id=${item.Id}` ``. **Không** hard-code
chuỗi path --- luôn qua `utilities/routers.ts`. - Thêm màn hình mới = 4
bước, thiếu bước nào là màn hình không chạy: 1. thêm entry vào
`routers.ts` (kèm `AppPermissionCode`), 2. thêm `case` trong
`renderComponent` của `hooks/blocks/newblock.tsx`, 3. thêm menu (backend
`Menus` nếu cần phân quyền, hoặc `defaultMenu` trong `newblock.tsx` nếu
không), 4. thêm key i18n cho **cả** `en/HR.json` và `vi/HR.json`.


## B6. Style

**MUST** - Component Fluent UI v9 từ `libapp/fluentv9`; icon từ
`@fluentui/react-icons` (`...Regular` / `...Filled`, ghép bằng
`bundleIcon` khi cần trạng thái). - Style theo thứ tự ưu tiên: 1.
Tailwind utility cho layout/spacing (`flex`, `gap-2`, `rounded-lg`,
`truncate`), 2. `useStyles` từ `hooks/blocks/styles.ts` cho style dùng
lại (`container`, `header`, `formCard`, `content`, dialog sizes), 3.
`makeStyles` cục bộ nếu style chỉ thuộc 1 component. - Không viết CSS
inline trừ giá trị động; dùng `tokens.*` thay vì màu hard-code khi có
token tương ứng. - Ngôn ngữ hình ảnh chuẩn (theo
`CountryDirectory.tsx`): Card nền trắng, `rounded-lg`/`rounded-xl`,
`shadow-sm` → hover `shadow-md`, accent blue-600/blue-50, success
green-500.

## B8. i18n

**MUST** - Không hard-code text hiển thị. Dùng
`const { t } = useTranslation(["HR"]);`. - Cấu trúc key:
`<Module>.<Nhóm>.<Key>` --- `General.Label.Name`,
`General.Notify.CreatedSuccess`, `Setting.CountryDirectory.Flag.Alt`. -
Label menu: top-level `Menu.<KeyKhôngDấuCách>`, con
`ChildMenu.<KeyKhôngDấuCách>`. - Mỗi key thêm mới **MUST** có mặt ở cả
`en/HR.json` và `vi/HR.json`. - Key `PascalCase`, không trùng phân biệt
hoa/thường (hiện `HR.json` đang có cả `Edit` và `EDIT` --- **không** tạo
thêm trường hợp như vậy).

------------------------------------------------------------------------

# PHẦN C --- Rule chung

**MUST** - Không commit secret (connection string, password, key). Dùng
`appsettings`/`extension.json` + biến môi trường. - BE và FE phải khớp
tên field DTO/model. Đổi DTO ở BE → sửa model FE trong cùng PR. - Trước
khi push: BE build sạch (`dotnet build`), FE `tsc --noEmit` không phát
sinh lỗi mới so với baseline. - Không để code chết: `//` code cũ,
`console.log`, biến không dùng. Cần giữ thì ghi rõ lý do + ticket. -
Comment giải thích **tại sao**, không mô tả lại code. Comment nhóm field
trong entity (`// ===== Classification =====`) thì được khuyến khích.

**Checklist review PR --- backend** - \[ \] Đủ 3 file
Command/Handler/Result trong thư mục riêng? -  \[ \] Mọi query
có `TenantId` + `ActiveFlag` + `IsActive`? Query đọc có
`AsNoTracking`? - \[ \] `cancellationToken` được truyền xuống hết?
`SaveChangesAsync` chỉ 1 lần? - \[ \] Lỗi nghiệp vụ trả Result thay vì
throw? - \[ \] Entity mới có Configuration + DbSet? - \[ \] Service mới
đã đăng ký trong `ConfigureServices.cs`? - \[ \] Function không chứa
business logic, không đụng DbContext?

**Checklist review PR --- frontend** - \[ \] Gọi API qua service, không
fetch trong component? - \[ \] Có `take(1)` + xử lý `error`?
Subscription được dọn? - \[ \] Model kế thừa
`BaseModel`/`CategoryEntity` và có `Init<X>`? - \[ \] Text đã i18n, có
đủ ở cả `en` và `vi`? - \[ \] Route qua `routers.ts` + đã thêm `case`
trong `newblock.tsx`? - \[ \] Quyền kiểm tra bằng `AppPermissionCode`,
không hard-code? - \[ \] Bộ file CRUD đặt tên đúng
`Main/List/Create/Edit/Delete/Form`?