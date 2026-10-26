# Noobietoria Studio — API Reference

## Quy ước tài liệu

| Ký hiệu | Ý nghĩa |
|---|---|
| 🖥️ **Server** | Chỉ gọi được trong `ServerScript` |
| 💻 **Client** | Chỉ gọi được trong `ClientScript` |
| 🔄 **Cả hai** | Gọi được từ cả `ServerScript` lẫn `ClientScript` |

- **GUID:** Tất cả tham số nhận tên Instance (`Instance`, `NpcName`, `SpawnName`, …) đều chấp nhận **GUID** thay thế (lấy từ `InstanceService.GetGUID`). Dùng GUID khi có nhiều Instance trùng tên.
- **Return type:** Ghi chú `-- → type` trong code block. Hàm không ghi chú trả về không có giá trị có ý nghĩa.
- **Callback:** Chữ ký đầy đủ được ghi trong phần "Lưu ý" của từng service.

---

## Hướng dẫn chọn service

### Mạng

| Khi cần… | Dùng |
|---|---|
| Tạo kênh giao tiếp Server ↔ Client qua Instance khai báo sẵn | [`NetworkEvent`](instance.md#networkevent) + [`NetworkService`](#networkservice) |
| Gọi API mạng hoàn toàn bằng code, không cần tạo Instance trước | [`NetworkEventService`](#networkeventservice) |

> **Tóm tắt:** `NetworkEvent` là Instance đặt trong Explorer; `NetworkService` điều khiển Instance đó bằng Luau; `NetworkEventService` là API mạng độc lập không cần Instance — ba thứ không thay thế lẫn nhau mà bổ sung cho nhau.

### Âm thanh

| Khi cần… | Dùng |
|---|---|
| Nhạc nền, âm thanh dài, có loop | [`AudioAssets`](#audioassets) (cấu hình) + [`AudioService`](#audioservice) (điều khiển) |
| Hiệu ứng âm thanh 3D tại vị trí trong không gian | [`SoundEffectService`](#soundeffectservice) |

### Kinh tế / Thanh toán

| Khi cần… | Dùng |
|---|---|
| Quản lý ví tiền, số dư của người chơi | [`CurrencyService`](#currencyservice) |
| Xử lý đơn hàng mua vật phẩm in-game | [`PurchaseService`](#purchaseservice) |
| Giao dịch trực tiếp giữa hai người chơi | [`TradingService`](#tradingservice) |
| Đăng bán / mua trên chợ Noobietoria | [`MarketplaceService`](#marketplaceservice) |

### Dịch chuyển / Hồi sinh

| Khi cần… | Dùng |
|---|---|
| Dịch chuyển người chơi trong cùng server | [`PlayerService.Teleport`](#playerservice) |
| Dịch chuyển giữa zone hoặc server khác | [`TeleportService`](#teleportservice) |
| Quản lý điểm spawn và hồi sinh | [`SpawnService`](#spawnservice) |

### Lọc văn bản

| Khi cần… | Dùng |
|---|---|
| Lọc tin nhắn theo cấp độ kênh chat | [`ChatService.SetFilter`](#chatservice) |
| Lọc văn bản hiển thị trong UI theo cài đặt từng người chơi | [`TextService.Filter`](#textservice) |

### DataStore — hai kiểu API

> `DataStoreService` cung cấp hai kiểu API song song; chọn một kiểu và dùng nhất quán:
>
> - **API gốc** (`Init / Insert / Get / Delete`): quản lý session tường minh, phù hợp khi cần kiểm soát kết nối chặt chẽ.
> - **API tương thích Roblox** (`GetDataStore / GetAsync / SetAsync`): tự quản lý kết nối, phù hợp khi port code từ Roblox.

---

## <img src="/Docs/studio/accessory.png" alt="Accessory icon" width="28" style="vertical-align:-6px" /> Accessory

> 🔄 **Cả hai** — cấu hình trong Explorer

Vật phẩm trang bị từ Noobietoria Marketplace gán trực tiếp cho Player hoặc NPC trong Explorer.

!!! note "Giới hạn"
    Accessory chỉ hỗ trợ **Items** và **Clothes**. Gamepasses và các loại asset khác không được chấp nhận dù có nhập ID.

**Cách triển khai:**

- **Player:** `[Tên Player] → IngameAccessoriesID` — nhập chuỗi ID, cách nhau bằng dấu `,`.
- **NPC:** `[Tên NPC] → IngameAccessoriesID` — nhập chuỗi ID, cách nhau bằng dấu `,`.

Không yêu cầu player đã mua item. Trường `DefaultAccessories` (root `Player`) phản ánh item player đã trang bị — không ghi đè trực tiếp; bật/tắt qua biến `DefaultAccessories`.

---

## <img src="/Docs/studio/accessoryservice.png" alt="AccessoryService icon" width="28" style="vertical-align:-6px" /> AccessoryService

> 🖥️ **Server**

Trang bị phụ kiện cho Player hoặc NPC bằng Luau runtime. Khác với `Accessory` (thiết lập tĩnh trong Explorer), `AccessoryService` cho phép trang bị động trong quá trình chơi.

**Lưu ý:** Tất cả 3 tham số đều bắt buộc, không được `nil`.

| Tham số | Kiểu | Mô tả |
|---|---|---|
| `Target` | `string` | Tên Player hoặc NPC |
| `TargetType` | `string` | `"Player"` hoặc `"NPC"` |
| `IDString` | `string` | Danh sách Asset ID, cách nhau bằng dấu `,` |

```luau
-- Trang bị phụ kiện cho player
AccessoryService("Player1", "Player", "1082345, 1082350, 1082360")

-- Trang bị phụ kiện cho NPC
AccessoryService("ShopKeeper", "NPC", "12345678, 87654321")
```

---

## <img src="/Docs/studio/audioassets.png" alt="AudioAssets icon" width="28" style="vertical-align:-6px" /> AudioAssets

> 🔄 **Cả hai** — cấu hình trong Explorer

Khai báo asset âm thanh để dùng trong game. Chỉ hỗ trợ file âm thanh (mp3, ogg, wav). Các loại asset khác không được chấp nhận.

**Cách tạo:**

1. `New → Audio`, vào `Audio → ID` và nhập Asset ID.
2. Bật loop: đặt `Audio → Loop = true`.

> Kết hợp với [`AudioService`](#audioservice) để phát/dừng/điều chỉnh bằng Luau.

---

## <img src="/Docs/studio/audiocservice.png" alt="AudioService icon" width="28" style="vertical-align:-6px" /> AudioService

> 🔄 **Cả hai**

Điều khiển runtime một Instance `Audio` đã được tạo từ [`AudioAssets`](#audioassets).

**Lưu ý:** `Name` là tên của Instance `Audio` trong Explorer (chuỗi, phân biệt hoa thường).

```luau
AudioService.New(Name, ID)           -- Tạo Audio mới với Asset ID và phát ngay
AudioService.Stop(Name)              -- Dừng và reset về đầu
AudioService.Resume(Name)            -- Tiếp tục sau Pause
AudioService.Pause(Name)             -- Tạm dừng
AudioService.SetVolume(Name, Volume) -- Volume: number (0.0 – 1.0)
AudioService.Change(Name, ID)        -- Đổi sang Asset ID khác
AudioService.Loop(Name, Boolean)     -- Bật/tắt loop
```

**Ví dụ:**

```luau
AudioService.New("BGM_Main", "12345678")
AudioService.SetVolume("BGM_Main", 0.7)
AudioService.Loop("BGM_Main", true)

-- Chuyển bài khi vào boss room
AudioService.Change("BGM_Main", "87654321")

-- Dừng hẳn khi thoát game
AudioService.Stop("BGM_Main")
```

---

## <img src="/Docs/studio/datastoreservice.png" alt="DataStoreService icon" width="28" style="vertical-align:-6px" /> DataStoreService

> 🖥️ **Server**

Lưu trữ và truy xuất dữ liệu bền vững từ cơ sở dữ liệu.

**Giới hạn:**

- Tối đa **15 datastores/game**.
- Key: tối đa **50 ký tự ASCII**; Value: tối đa **1 MB** (phải là JSON hợp lệ).
- Dung lượng: **25 MB + (5 MB × N)** trong đó N là tổng số visits.
- Rate limit: **15 + (10 × CCU)** request/server/phút; tối đa **16 kết nối đồng thời**.
- Không có yêu cầu trong **15 giây** → tự đóng kết nối (trừ khi có data đang xử lý dở).

**API gốc** — quản lý session tường minh:

```luau
DataStoreService.Init(SessionID)               -- Mở kết nối; SessionID: chuỗi base32 do bạn tự tạo
DataStoreService.Close(SessionID)              -- Đóng kết nối; luôn gọi sau khi xong
DataStoreService.Insert(SessionID, Key, Value) -- Ghi/cập nhật; Value phải là chuỗi JSON
DataStoreService.Get(SessionID, Key)           -- → string (JSON) | nil
DataStoreService.GetKeys(SessionID)            -- → table (danh sách key)
DataStoreService.Delete(SessionID, Key)        -- Xóa một key
```

**API tương thích Roblox** — tự quản lý kết nối:

```luau
DataStoreService:GetDataStore(name, scope)                   -- → DataStore
DataStoreService:GetOrderedDataStore(name, scope)            -- → OrderedDataStore
DataStoreService:GetRequestBudgetForRequestType(requestType) -- → number (request còn lại trong phút này)

DataStore:GetAsync(key)                           -- → value | nil
DataStore:SetAsync(key, value, userIds, options)  -- userIds: {UserId} (cho GDPR); options: DataStoreSetOptions
DataStore:UpdateAsync(key, transformFunction)     -- transformFunction(oldValue) → newValue
DataStore:RemoveAsync(key)                        -- → oldValue | nil
```

!!! warning "Bảo mật"
    Không tin tưởng dữ liệu từ client khi ghi vào DataStore. Luôn xác thực và làm sạch dữ liệu phía server trước khi gọi `Insert` / `SetAsync`.

---

## <img src="/Docs/studio/tweenservice.png" alt="TweenService icon" width="28" style="vertical-align:-6px" /> TweenService

> 🔄 **Cả hai**

Di chuyển, xoay hoặc thay đổi thuộc tính của Part/Model mượt mà theo thời gian.

**Lưu ý:** `Instance` chấp nhận tên hoặc GUID (dùng GUID khi có nhiều Part trùng tên).

**Giá trị hợp lệ:**

- `EasingDirection`: `"In"` | `"Out"` | `"InOut"`
- `EasingStyle`: `"Linear"` | `"Quad"` | `"Cubic"` | `"Quart"` | `"Quint"` | `"Sine"` | `"Bounce"` | `"Elastic"` | `"Back"` | `"Exponential"` | `"Circular"`
- `RGBA`: `{R, G, B, A}` với giá trị `0–255`; truyền `nil` để giữ nguyên màu hiện tại.

```luau
TweenService.Move(Instance, EndPosition, Time, EasingDirection, EasingStyle, Loop, RGBA)
             -- EndPosition: {X, Y, Z}; Time: number (giây); Loop: bool
TweenService.Stop(Instance)    -- Dừng tween, giữ nguyên vị trí hiện tại
TweenService.Resume(Instance)  -- Tiếp tục sau Pause
TweenService.Pause(Instance)   -- Tạm dừng
TweenService.Back(Instance)    -- Reset về vị trí/trạng thái ban đầu
```

---

## <img src="/Docs/studio/playerservice.png" alt="PlayerService icon" width="28" style="vertical-align:-6px" /> PlayerService

> 🖥️ **Server**

Tra cứu thông tin và thao tác với người chơi đang kết nối.

```luau
PlayerService.GetPlayers()                   -- → table (danh sách Username)
PlayerService.GetPlayer(Username)            -- → PlayerObject | nil
PlayerService.GetPlayerName(UserId)          -- → string (Username) | nil
PlayerService.GetPlayerCountry(UserId)       -- → string (ISO 3166-1 alpha-2, ví dụ "VN") | nil
PlayerService.GetPlayerAvatar(UserId)        -- → ImageObject | nil
PlayerService.GetPlayerAvatarUrl(UserId)     -- → string (URL) | nil
PlayerService.GetPlayerThumbnail(UserId)     -- → ImageObject | nil
PlayerService.GetPlayerThumbnailUrl(UserId)  -- → string (URL) | nil
PlayerService.GetPlayerCharacter(UserId)     -- → CharacterObject | nil
PlayerService.GetPlayerCharacterUrl(UserId)  -- → string (URL) | nil
PlayerService.GetCoords(Username)            -- → {X, Y, Z} | nil
PlayerService.Teleport(Username, Coords)     -- Dịch chuyển trong cùng server; Coords: {X, Y, Z}
```

> **Xem thêm:** [`TeleportService`](#teleportservice) để dịch chuyển giữa zone/server; [`SpawnService`](#spawnservice) để quản lý điểm hồi sinh.

---

## <img src="/Docs/studio/httpservice.png" alt="HttpService icon" width="28" style="vertical-align:-6px" /> HttpService

> 🖥️ **Server**

Gửi HTTP request đến API bên ngoài.

**Methods hỗ trợ:** `GET` | `POST` | `PUT` | `DELETE` | `HEAD` | `OPTIONS`

**ContentType hỗ trợ:** `"form"` (URL-encoded) | `"json"` | `"multipart"` (upload file)

```luau
HttpService.Prepare(
    Id: number,          -- ID tự chọn để tham chiếu request này
    Method: string,
    Url: string,         -- URL đầy đủ (https://...)
    Headers: table,      -- { ["Header-Name"] = "value", ... }
    Body: string,        -- Nội dung body
    ContentType: string
)
HttpService.Request(Id)  -- → { StatusCode: number, Body: string, Headers: table }
HttpService.Release(Id)  -- Giải phóng bộ nhớ sau khi xong
```

**Ví dụ:**

```luau
HttpService.Prepare(1, "POST", "https://api.example.com/log",
    { ["Authorization"] = "Bearer mytoken" },
    '{"event":"login","user":"Player1"}',
    "json"
)
local res = HttpService.Request(1)
if res.StatusCode == 200 then
    -- xử lý response
end
HttpService.Release(1)
```

!!! warning "Bảo mật"
    Không ghi API key hoặc token vào `ClientScript`. Luôn gọi `HttpService` từ `ServerScript`.

---

## <img src="/Docs/studio/uiservice.png" alt="UIService icon" width="28" style="vertical-align:-6px" /> UIService

> 💻 **Client**

Tạo và quản lý giao diện người dùng (UI) trong game. Hỗ trợ: `Frame`, `TextLabel`, `TextButton`, `ImageLabel`, `ImageButton`, `TextBox`, `ScrollingFrame`.

**Lưu ý chung:**

- `Name`: tên định danh element (chuỗi, không được trùng lặp trong toàn bộ UI).
- `Parent`: tên element cha hoặc `"ScreenGui"` nếu là root.
- `Properties`: table các thuộc tính (xem danh sách thuộc tính hỗ trợ theo loại element tại [Instance Reference](instance.md)).
- `AssetId`: ID hình ảnh từ NImageAssets (chuỗi), dùng cho `ImageLabel` và `ImageButton`.
- `SetCanvasSize`, `GetCanvasSize`, `SetScrollPosition`, `GetScrollPosition`: chỉ áp dụng cho `ScrollingFrame`.
- `SetPlaceholder`: chỉ áp dụng cho `TextBox`.

```luau
UIService.Create(Name, Type, Parent, Properties)   -- Tạo element mới
UIService.Destroy(Name)                            -- Xóa element
UIService.SetProperty(Name, Property, Value)       -- Đặt thuộc tính
UIService.GetProperty(Name, Property)              -- → value
UIService.SetVisible(Name, Boolean)
UIService.Tween(Name, Properties, Time, EasingDirection, EasingStyle)
UIService.OnClick(Name, Callback)                  -- Callback: function()
UIService.OnTextChanged(Name, Callback)            -- Callback: function(newText: string)
UIService.RemoveCallback(Name)
UIService.SetText(Name, Text)
UIService.GetText(Name)                            -- → string
UIService.SetImage(Name, AssetId)
UIService.SetPlaceholder(Name, PlaceholderText)
UIService.SetCanvasSize(Name, Width, Height)
UIService.GetCanvasSize(Name)                      -- → { Width: number, Height: number }
UIService.SetScrollPosition(Name, X, Y)
UIService.GetScrollPosition(Name)                  -- → { X: number, Y: number }
```

---

## <img src="/Docs/studio/instanceservice.png" alt="InstanceService icon" width="28" style="vertical-align:-6px" /> InstanceService

> 🔄 **Cả hai**

Tạo, xóa, di chuyển và truy vấn [Instance](instance.md) (Part, Model, NetworkEvent, Script, …) trong Workspace hoặc các vùng chứa khác.

**Lưu ý:**

- `Type`: `"Part"` | `"Model"` | `"SpawnLocation"` | `"NetworkEvent"` | `"ServerScript"` | `"ClientScript"` | `"ModuleScript"` | … (xem [Instance Reference](instance.md)).
- `Parent`: `"Workspace"` | `"ReplicatedStorage"` | `"ServerStorage"` | `"StarterPlayerScripts"` | …
- `GUID`: định danh duy nhất, dùng thay `Name + Index` khi có nhiều Instance trùng tên.
- `Index`: số thứ tự (bắt đầu từ `1`) theo thứ tự tạo trong danh sách Instance trùng tên. Mặc định `1` khi bỏ qua.
- Với `"ServerScript"` / `"ClientScript"`: truyền mã Luau qua `Properties.Source`; `Properties.Enabled` mặc định `true`.
- Với `"ModuleScript"`: `Properties.Source` **phải** có `return` một giá trị; nếu không sẽ lỗi khi `require`.

```luau
InstanceService.Create(Name, Type, Parent, Properties)  -- Tạo Instance mới
InstanceService.Destroy(Name, Index)                    -- Xóa theo tên (và Index tùy chọn)
InstanceService.DestroyByGUID(GUID)                     -- Xóa chính xác theo GUID
InstanceService.GetGUID(Name, Index)                    -- → string (GUID) | nil
InstanceService.GetGUIDs(Name)                          -- → table (tất cả GUID trùng tên)
InstanceService.GetByGUID(GUID)                         -- → InstanceObject | nil
InstanceService.Move(Name, Position, Index)             -- Di chuyển; Position: {X, Y, Z}
InstanceService.MoveByGUID(GUID, Position)
InstanceService.SetEnabled(Name, Boolean, Index)        -- Bật/tắt Script
InstanceService.IsEnabled(Name, Index)                  -- → bool
InstanceService.Require(Name, Index)                    -- → giá trị return của ModuleScript
InstanceService.RequireByGUID(GUID)                     -- → giá trị return của ModuleScript
```

---

## <img src="/Docs/studio/networkservice.png" alt="NetworkService icon" width="28" style="vertical-align:-6px" /> NetworkService

> 🖥️ **Server** (FireClient, FireAllClients, OnServerEvent) / 💻 **Client** (FireServer, OnClientEvent) / 🔄 (Create, Destroy, Disconnect)

Điều khiển Instance [`NetworkEvent`](instance.md#networkevent) để giao tiếp hai chiều giữa [`ServerScript`](instance.md#serverscript) và [`ClientScript`](instance.md#clientscript).

**Lưu ý:**

- `Name`: tên của `NetworkEvent` Instance (phải khớp ở cả Server và Client).
- `Index`: dùng khi có nhiều `NetworkEvent` trùng tên. Mặc định `1`. Có thể thay bằng GUID.
- `Username`: tên người chơi nhận event; truyền `"*"` để gửi cho tất cả client.
- `...`: dữ liệu tùy ý, phải serializable: `string`, `number`, `boolean`, `table` thuần.

!!! warning "Tham số Index với variadic"
    `FireClient(Name, Username, ..., Index)` — khi cần chỉ định `Index` (nhiều NetworkEvent trùng tên), `Index` phải là tham số cuối và là kiểu `number`. Hệ thống phân biệt `Index` và dữ liệu dựa vào vị trí cuối và kiểu số nguyên.

```luau
-- Gọi được ở cả hai phía
NetworkService.Create(Name, Parent)          -- Tạo Instance NetworkEvent
NetworkService.Destroy(Name, Index)          -- Xóa NetworkEvent
NetworkService.Disconnect(Name, Index)       -- Ngắt listener

-- Chỉ gọi trong ServerScript
NetworkService.FireClient(Name, Username, ..., Index)    -- Gửi đến một client
NetworkService.FireAllClients(Name, ..., Index)          -- Gửi đến tất cả client
NetworkService.OnServerEvent(Name, Callback, Index)      -- Callback: function(Username: string, ...)

-- Chỉ gọi trong ClientScript
NetworkService.FireServer(Name, ..., Index)              -- Gửi lên server
NetworkService.OnClientEvent(Name, Callback, Index)      -- Callback: function(...)
```

---

## <img src="/Docs/studio/collisionservice.png" alt="CollisionService icon" width="28" style="vertical-align:-6px" /> CollisionService

> 🖥️ **Server**

Quản lý va chạm giữa các đối tượng và nhóm va chạm.

**Lưu ý:** `Instance` chấp nhận tên hoặc GUID.

```luau
CollisionService.CreateGroup(GroupName)                  -- Tạo nhóm va chạm mới
CollisionService.DeleteGroup(GroupName)                  -- Xóa nhóm
CollisionService.AssignGroup(Instance, GroupName)        -- Gán Instance vào nhóm
CollisionService.SetCollision(GroupA, GroupB, Boolean)   -- Bật/tắt va chạm giữa hai nhóm
CollisionService.GetGroup(Instance)                      -- → string (GroupName) | nil
CollisionService.GetGroups()                             -- → table (danh sách GroupName)
```

---

## <img src="/Docs/studio/lightingservice.png" alt="LightingService icon" width="28" style="vertical-align:-6px" /> LightingService

> 🖥️ **Server** *(alias: EnvironmentService)*

Điều chỉnh thời gian trong ngày, ánh sáng môi trường và thời tiết.

**Giá trị hợp lệ:**

- `TimeOfDay`: chuỗi định dạng `"HH:MM:SS"` (ví dụ `"18:30:00"`).
- `WeatherType`: `"Clear"` | `"Rain"` | `"Storm"` | `"Fog"` | `"Snow"`.
- `Intensity`: `number` (`0.0` – `1.0`).
- `RGBA`: `{R, G, B, A}` với giá trị `0–255`.
- `Properties`: table các thuộc tính ánh sáng (Brightness, ClockTime, GeographicLatitude, …).

```luau
LightingService.SetTime(TimeOfDay)                -- Đặt giờ trong ngày
LightingService.GetTime()                         -- → string ("HH:MM:SS")
LightingService.SetWeather(WeatherType, Intensity)
LightingService.GetWeather()                      -- → { Type: string, Intensity: number }
LightingService.SetProperty(Property, Value)
LightingService.GetProperty(Property)             -- → value
LightingService.SetAmbient(RGBA)
LightingService.SetFog(Enabled, Color, Start, End)
             -- Color: {R,G,B,A}; Start/End: number (studs)
```

---

## <img src="/Docs/studio/notificationservice.png" alt="NotificationService icon" width="28" style="vertical-align:-6px" /> NotificationService

> 🖥️ **Server**

Hiển thị thông báo toast, popup hoặc banner cho người chơi.

**Giá trị hợp lệ:**

- `Type`: `"Toast"` | `"Popup"` | `"Banner"`.
- `Username`: tên người chơi; truyền `"*"` để gửi cho tất cả.
- `Duration`: `number` (giây).

```luau
NotificationService.Send(Username, Title, Message, Type, Duration)
NotificationService.SendAll(Title, Message, Type, Duration)
NotificationService.Dismiss(Username, NotificationId)
NotificationService.OnDismiss(Username, Callback)
             -- Callback: function(Username: string, NotificationId: string)
```

---

## <img src="/Docs/studio/inventoryservice.png" alt="InventoryService icon" width="28" style="vertical-align:-6px" /> InventoryService

> 🖥️ **Server**

Quản lý túi đồ của người chơi.

**Lưu ý:**

- `ItemId`: ID định danh vật phẩm (chuỗi, tự định nghĩa trong game).
- `Quantity`: số nguyên ≥ 1.
- `Metadata`: thông tin bổ sung tùy ý (table, có thể nil).

```luau
InventoryService.GetInventory(Username)                        -- → table ({ ItemId, Quantity, Metadata }[])
InventoryService.AddItem(Username, ItemId, Quantity, Metadata)
InventoryService.RemoveItem(Username, ItemId, Quantity)
InventoryService.HasItem(Username, ItemId)                     -- → bool
InventoryService.GetItemCount(Username, ItemId)                -- → number
InventoryService.ClearInventory(Username)
InventoryService.TransferItem(FromUsername, ToUsername, ItemId, Quantity)
```

!!! warning "Bảo mật"
    Không gọi `AddItem` / `RemoveItem` dựa trực tiếp vào dữ liệu từ `ClientScript`. Luôn xác thực logic phía `ServerScript` trước.

---

## <img src="/Docs/studio/leaderboardservice.png" alt="LeaderboardService icon" width="28" style="vertical-align:-6px" /> LeaderboardService

> 🖥️ **Server**

Quản lý bảng xếp hạng và điểm số.

**Giá trị hợp lệ:**

- `Order`: `"Ascending"` (tăng dần, điểm thấp = xếp cao) | `"Descending"` (giảm dần, điểm cao = xếp cao).

```luau
LeaderboardService.CreateBoard(BoardName, Order)
LeaderboardService.DeleteBoard(BoardName)
LeaderboardService.SetScore(BoardName, Username, Score)    -- Score: number
LeaderboardService.GetScore(BoardName, Username)           -- → number | nil
LeaderboardService.AddScore(BoardName, Username, Delta)    -- Delta: number (có thể âm)
LeaderboardService.GetTopPlayers(BoardName, Limit)         -- → { { Username, Score } }[] (sorted)
LeaderboardService.GetRank(BoardName, Username)            -- → number | nil
LeaderboardService.ResetBoard(BoardName)
```

---

## <img src="/Docs/studio/spawnservice.png" alt="SpawnService icon" width="28" style="vertical-align:-6px" /> SpawnService

> 🖥️ **Server**

Quản lý điểm hồi sinh (spawn point) của Player và NPC.

**Giá trị hợp lệ:**

- `TargetType`: `"Player"` | `"NPC"`.

```luau
SpawnService.CreateSpawn(SpawnName, Position)             -- Position: {X, Y, Z}
SpawnService.DeleteSpawn(SpawnName)
SpawnService.SetSpawn(SpawnName, Position)
SpawnService.GetSpawn(SpawnName)                          -- → { Name, Position } | nil
SpawnService.GetAllSpawns()                               -- → table
SpawnService.AssignSpawn(TargetType, Username, SpawnName) -- Gán spawn mặc định
SpawnService.RespawnAt(TargetType, Username, SpawnName)   -- Hồi sinh tại spawn cụ thể
SpawnService.Respawn(TargetType, Username)                -- Hồi sinh tại spawn đã gán
```

---

## <img src="/Docs/studio/cameraservice.png" alt="CameraService icon" width="28" style="vertical-align:-6px" /> CameraService

> 🖥️ **Server** *(hiệu ứng được áp dụng phía client của từng người chơi)*

Điều khiển góc nhìn camera và rung màn hình. Các hàm nhận `Username` được thực thi trên client tương ứng; truyền `"*"` để áp dụng cho tất cả người chơi.

**Giá trị hợp lệ:**

- `CameraType`: `"Classic"` | `"Follow"` | `"Fixed"` | `"Scriptable"`.

```luau
CameraService.SetType(Username, CameraType)
CameraService.SetPosition(Username, Position, LookAt)  -- Position, LookAt: {X, Y, Z}
CameraService.SetFOV(Username, FieldOfView)            -- FieldOfView: number (độ)
CameraService.LockTo(Username, Instance)               -- Instance: tên hoặc GUID
CameraService.Unlock(Username)
CameraService.Shake(Username, ShakeIntensity, Duration) -- ShakeIntensity, Duration: number
CameraService.Reset(Username)                          -- Khôi phục camera mặc định
```

---

## <img src="/Docs/studio/analyticsservice.png" alt="AnalyticsService icon" width="28" style="vertical-align:-6px" /> AnalyticsService

> 🖥️ **Server**

Theo dõi sự kiện và chỉ số trong game.

**Lưu ý:**

- `EventName`: tên sự kiện (chuỗi, ví dụ `"purchase"`, `"level_up"`).
- `Properties`: thông tin bổ sung (table, có thể nil).
- `Username`: tên người chơi liên quan (chuỗi, có thể nil nếu là sự kiện hệ thống).

```luau
AnalyticsService.TrackEvent(EventName, Username, Properties)
AnalyticsService.TrackError(ErrorMessage, Username, Properties)
AnalyticsService.SetCounter(MetricName, Value)         -- Value: number
AnalyticsService.IncrementCounter(MetricName, Delta)   -- Delta: number
AnalyticsService.GetCounter(MetricName)                -- → number
AnalyticsService.GetEvents(EventName, Limit)           -- → table
AnalyticsService.ClearEvents(EventName)
```

---

## <img src="/Docs/studio/achievementservice.png" alt="AchievementService icon" width="28" style="vertical-align:-6px" /> AchievementService

> 🖥️ **Server**

Quản lý và mở khóa thành tựu cho người chơi.

**Lưu ý:**

- `MaxProgress`: ngưỡng tiến trình để tự động mở khóa thành tựu.
- `Progress` / `Delta`: number (≥ 0).

```luau
AchievementService.Register(AchievementId, Name, Description, MaxProgress)
AchievementService.Unregister(AchievementId)
AchievementService.GetAll()                                      -- → table
AchievementService.GetProgress(Username, AchievementId)          -- → number
AchievementService.SetProgress(Username, AchievementId, Progress)
AchievementService.AddProgress(Username, AchievementId, Delta)
AchievementService.Unlock(Username, AchievementId)               -- Mở khóa thủ công
AchievementService.IsUnlocked(Username, AchievementId)           -- → bool
AchievementService.GetUnlocked(Username)                         -- → table (AchievementId[])
AchievementService.OnUnlock(Username, Callback)
             -- Callback: function(Username: string, AchievementId: string)
```

---

## <img src="/Docs/studio/purchaseservice.png" alt="PurchaseService icon" width="28" style="vertical-align:-6px" /> PurchaseService

> 🖥️ **Server**

Xử lý đơn hàng và giao dịch mua vật phẩm trong game.

**Lưu ý:**

- `OrderId`: được **tự sinh** bởi hệ thống khi gọi `CreateOrder`. Dùng giá trị trả về để gọi các hàm tiếp theo (`ConfirmOrder`, `CancelOrder`, …).
- `CurrencyType`: tên loại tiền tệ (chuỗi, ví dụ `"Coin"`, `"Gem"`).

```luau
PurchaseService.CreateOrder(Username, ItemId, Amount, CurrencyType) -- → OrderId: string
PurchaseService.ConfirmOrder(OrderId)
PurchaseService.CancelOrder(OrderId)
PurchaseService.GetOrder(OrderId)              -- → OrderObject | nil
PurchaseService.GetOrderHistory(Username, Limit) -- → table
PurchaseService.Refund(OrderId)
PurchaseService.OnPurchase(Username, Callback)
             -- Callback: function(Username: string, OrderId: string, ItemId: string)
```

**Ví dụ:**

```luau
local orderId = PurchaseService.CreateOrder("Player1", "Sword_01", 1, "Coin")
-- kiểm tra điều kiện trước khi xác nhận
PurchaseService.ConfirmOrder(orderId)
```

!!! warning "Bảo mật"
    Luôn xác nhận đơn hàng phía server. Không để client gọi `ConfirmOrder` trực tiếp.

---

## <img src="/Docs/studio/chatservice.png" alt="ChatService icon" width="28" style="vertical-align:-6px" /> ChatService

> 🖥️ **Server**

Quản lý kênh chat, bong bóng chat và lọc từ ngữ.

**Giá trị hợp lệ:**

- `FilterLevel`: `"None"` | `"Moderate"` | `"Strict"`.

```luau
ChatService.CreateChannel(ChannelName)
ChatService.DeleteChannel(ChannelName)
ChatService.JoinChannel(Username, ChannelName)
ChatService.LeaveChannel(Username, ChannelName)
ChatService.SendMessage(Username, ChannelName, Message)
ChatService.MutePlayer(Username, Duration)     -- Duration: number (giây); -1 để mute vĩnh viễn
ChatService.UnmutePlayer(Username)
ChatService.SetFilter(ChannelName, FilterLevel)
ChatService.ShowBubble(Username, Message, Duration)
ChatService.HideBubble(Username)
ChatService.OnMessage(ChannelName, Callback)
             -- Callback: function(Username: string, Message: string)
```

---

## <img src="/Docs/studio/badgeservice.png" alt="BadgeService icon" width="28" style="vertical-align:-6px" /> BadgeService

> 🖥️ **Server**

Cấp và quản lý huy hiệu / danh hiệu cho người chơi.

```luau
BadgeService.Register(BadgeId, Name, Description)
BadgeService.Unregister(BadgeId)
BadgeService.Award(Username, BadgeId)
BadgeService.Revoke(Username, BadgeId)
BadgeService.HasBadge(Username, BadgeId)    -- → bool
BadgeService.GetBadges(Username)            -- → table (BadgeId[])
BadgeService.GetAll()                       -- → table
BadgeService.OnAward(Username, Callback)
             -- Callback: function(Username: string, BadgeId: string)
```

---

## <img src="/Docs/studio/currencyservice.png" alt="CurrencyService icon" width="28" style="vertical-align:-6px" /> CurrencyService

> 🖥️ **Server**

Quản lý ví tiền và các loại tiền tệ của người chơi.

**Lưu ý:**

- `CurrencyType`: tên loại tiền tệ (chuỗi, ví dụ `"Coin"`, `"Gem"`, `"EventToken"`). Tự định nghĩa trong game.
- `Amount`: number ≥ 0.

```luau
CurrencyService.GetBalance(Username, CurrencyType)              -- → number
CurrencyService.Add(Username, CurrencyType, Amount)
CurrencyService.Deduct(Username, CurrencyType, Amount)
CurrencyService.Transfer(FromUsername, ToUsername, CurrencyType, Amount)
CurrencyService.SetBalance(Username, CurrencyType, Amount)
CurrencyService.GetAllBalances(Username)                        -- → table ({ CurrencyType: Amount })
CurrencyService.OnTransaction(Username, Callback)
             -- Callback: function(Username: string, CurrencyType: string, Delta: number, NewBalance: number)
```

!!! warning "Bảo mật"
    Không cho phép `ClientScript` gọi `Add` hoặc `Transfer` trực tiếp. Mọi thay đổi số dư phải đi qua logic xác thực phía `ServerScript`.

---

## <img src="/Docs/studio/questservice.png" alt="QuestService icon" width="28" style="vertical-align:-6px" /> QuestService

> 🖥️ **Server**

Quản lý tiến trình nhiệm vụ và phần thưởng.

**Lưu ý:**

- `Steps`: table mô tả các bước nhiệm vụ, mỗi bước gồm `{ name, maxProgress }`.
- `Rewards`: table phần thưởng khi hoàn thành (định dạng tự do, hệ thống lưu và trả lại nguyên vẹn).
- `StepIndex`: số thứ tự bước (bắt đầu từ `1`).

```luau
QuestService.Register(QuestId, Name, Description, Steps, Rewards)
QuestService.Unregister(QuestId)
QuestService.Assign(Username, QuestId)
QuestService.Unassign(Username, QuestId)
QuestService.GetProgress(Username, QuestId)                      -- → table (progress từng bước)
QuestService.AddProgress(Username, QuestId, StepIndex, Delta)
QuestService.SetProgress(Username, QuestId, StepIndex, Progress)
QuestService.Complete(Username, QuestId)                         -- Hoàn thành thủ công
QuestService.IsCompleted(Username, QuestId)                      -- → bool
QuestService.GetActive(Username)                                 -- → table (QuestId[])
QuestService.OnComplete(Username, Callback)
             -- Callback: function(Username: string, QuestId: string)
```

---

## <img src="/Docs/studio/rewardservice.png" alt="RewardService icon" width="28" style="vertical-align:-6px" /> RewardService

> 🖥️ **Server**

Quản lý quà điểm danh, quà hàng ngày và quà mã code.

**Lưu ý:**

- `Rewards`: table phần thưởng (định dạng tự do, trả lại nguyên vẹn cho `OnClaim`).
- `MaxUses`: số lần đổi tối đa (`-1` để không giới hạn).
- `ExpiresAt`: timestamp Unix (number, giây) hoặc `nil` để không hết hạn.

```luau
RewardService.ClaimDaily(Username)
RewardService.GetDailyStreak(Username)                           -- → number
RewardService.ClaimLoginReward(Username, RewardId)
RewardService.RedeemCode(Username, Code)                         -- → bool (thành công hay không)
RewardService.RegisterCode(Code, Rewards, MaxUses, ExpiresAt)
RewardService.RevokeCode(Code)
RewardService.GetCodeInfo(Code)                                  -- → { Rewards, MaxUses, Uses, ExpiresAt } | nil
RewardService.OnClaim(Username, Callback)
             -- Callback: function(Username: string, RewardId: string, Rewards: table)
```

---

## <img src="/Docs/studio/banservice.png" alt="BanService icon" width="28" style="vertical-align:-6px" /> BanService

> 🖥️ **Server** *(alias: PunishService)*

Cấm, cảnh báo hoặc kick người chơi vi phạm.

**Lưu ý:**

- `Duration`: thời gian ban tính bằng giây; truyền `-1` để ban vĩnh viễn.

```luau
BanService.Kick(Username, Reason)
BanService.Ban(Username, Reason, Duration)
BanService.Unban(Username)
BanService.IsBanned(Username)                    -- → bool
BanService.GetBanInfo(Username)                  -- → { Reason, Duration, BannedAt } | nil
BanService.GetBanHistory(Username, Limit)        -- → table
BanService.Warn(Username, Reason)
BanService.GetWarnings(Username)                 -- → table ({ Reason, WarnedAt }[])
BanService.ClearWarnings(Username)
```

---

## <img src="/Docs/studio/matchmakingservice.png" alt="MatchmakingService icon" width="28" style="vertical-align:-6px" /> MatchmakingService

> 🖥️ **Server**

Xếp hàng, ghép trận và phân chia phòng chơi.

```luau
MatchmakingService.CreateQueue(QueueName, MaxPlayers)
MatchmakingService.DeleteQueue(QueueName)
MatchmakingService.Join(Username, QueueName)
MatchmakingService.Leave(Username, QueueName)
MatchmakingService.GetQueue(QueueName)            -- → table (Username[])
MatchmakingService.CreateRoom(RoomId, QueueName)
MatchmakingService.CloseRoom(RoomId)
MatchmakingService.GetRoom(RoomId)                -- → { RoomId, QueueName, Players: table } | nil
MatchmakingService.GetRooms(QueueName)            -- → table
MatchmakingService.OnMatch(QueueName, Callback)
             -- Callback: function(RoomId: string, Players: table)
```

---

## <img src="/Docs/studio/partyservice.png" alt="PartyService icon" width="28" style="vertical-align:-6px" /> PartyService

> 🖥️ **Server**

Tạo nhóm, mời bạn bè và tổ đội cùng chơi.

```luau
PartyService.CreateParty(Leader, MaxSize)         -- → PartyId: string
PartyService.DisbandParty(PartyId)
PartyService.Invite(PartyId, Username)
PartyService.AcceptInvite(PartyId, Username)
PartyService.DeclineInvite(PartyId, Username)
PartyService.Kick(PartyId, Username)
PartyService.Leave(PartyId, Username)
PartyService.GetParty(PartyId)                    -- → { Leader, Members: table, MaxSize } | nil
PartyService.GetPartyOf(Username)                 -- → PartyId | nil
PartyService.TransferLeader(PartyId, Username)
PartyService.OnInvite(Username, Callback)
             -- Callback: function(Username: string, PartyId: string, InvitedBy: string)
```

---

## <img src="/Docs/studio/dialogueservice.png" alt="DialogueService icon" width="28" style="vertical-align:-6px" /> DialogueService

> 🖥️ **Server**

Xử lý hội thoại và lựa chọn câu trả lời khi tương tác với NPC.

**Lưu ý:**

- `Nodes`: table các nút hội thoại, mỗi nút gồm: `{ text: string, choices: string[], callback: function? }`.

```luau
DialogueService.Register(DialogueId, NpcName, Nodes)
DialogueService.Unregister(DialogueId)
DialogueService.Start(Username, DialogueId)
DialogueService.End(Username)
DialogueService.GetCurrent(Username)              -- → { DialogueId, NodeIndex } | nil
DialogueService.Choose(Username, ChoiceIndex)
DialogueService.OnStart(NpcName, Callback)
             -- Callback: function(Username: string, DialogueId: string)
DialogueService.OnEnd(NpcName, Callback)
             -- Callback: function(Username: string, DialogueId: string)
DialogueService.OnChoice(DialogueId, Callback)
             -- Callback: function(Username: string, ChoiceIndex: number)
```

---

## <img src="/Docs/studio/teleportservice.png" alt="TeleportService icon" width="28" style="vertical-align:-6px" /> TeleportService

> 🖥️ **Server**

Dịch chuyển người chơi giữa zone hoặc server.

**Lưu ý:**

- `ServerId`: ID server đích (chuỗi); có thể nil để dùng server mặc định của zone.
- `Position`: `{X, Y, Z}` điểm xuất hiện sau khi dịch chuyển (có thể nil để dùng spawn mặc định).

```luau
TeleportService.TeleportToZone(Username, ZoneName, Position)
TeleportService.TeleportToServer(Username, ServerId, Position)
TeleportService.TeleportGroup(Usernames, ZoneName, Position)   -- Usernames: table (string[])
TeleportService.RegisterZone(ZoneName, ServerId, SpawnPosition)
TeleportService.UnregisterZone(ZoneName)
TeleportService.GetZone(ZoneName)             -- → { ZoneName, ServerId, SpawnPosition } | nil
TeleportService.GetZones()                    -- → table
TeleportService.OnTeleport(Username, Callback)
             -- Callback: function(Username: string, ZoneName: string)
```

---

## <img src="/Docs/studio/cooldownservice.png" alt="CooldownService icon" width="28" style="vertical-align:-6px" /> CooldownService

> 🔄 **Cả hai**

Quản lý thời gian hồi (cooldown) cho kỹ năng, item và hành động.

**Lưu ý:**

- `CooldownId`: ID tự định nghĩa cho từng loại cooldown (ví dụ: `"Skill_Fireball"`, `"Item_Potion"`).
- `Duration`: số giây (number > 0).

```luau
CooldownService.Start(Username, CooldownId, Duration)
CooldownService.Cancel(Username, CooldownId)
CooldownService.IsReady(Username, CooldownId)          -- → bool (true nếu hết cooldown)
CooldownService.GetRemaining(Username, CooldownId)     -- → number (giây còn lại; 0 nếu sẵn sàng)
CooldownService.GetAll(Username)                       -- → table ({ CooldownId: Remaining })
CooldownService.Reset(Username, CooldownId)            -- Đặt lại về 0 (sẵn sàng ngay)
CooldownService.OnReady(Username, CooldownId, Callback)
             -- Callback: function(Username: string, CooldownId: string)
```

---

## <img src="/Docs/studio/effectservice.png" alt="EffectService icon" width="28" style="vertical-align:-6px" /> EffectService

> 🖥️ **Server** *(hiệu ứng màn hình được áp dụng phía client của từng người chơi)*

Tạo hiệu ứng hình ảnh, particle và hiệu ứng màn hình.

**Lưu ý:**

- `EffectId`: ID định danh asset hiệu ứng (chuỗi).
- `Instance`: tên hoặc GUID của Part/Model để gắn hiệu ứng lên (có thể nil khi dùng `Position`).
- `Username`: người chơi cần áp dụng hiệu ứng màn hình; truyền `"*"` cho tất cả.
- `EffectType`: `"Blur"` | `"ColorGrade"` | `"Flash"` | `"Vignette"` | `"Chromatic"`.

```luau
EffectService.Emit(EffectId, Position, Properties)     -- Phát hiệu ứng tại vị trí; Position: {X, Y, Z}
EffectService.EmitOn(EffectId, Instance, Properties)   -- Gắn hiệu ứng lên Instance
EffectService.Stop(EffectId)
EffectService.StopAll()
EffectService.ScreenEffect(Username, EffectType, Properties, Duration)
EffectService.ClearScreenEffect(Username)
```

---

## <img src="/Docs/studio/soundeffectservice.png" alt="SoundEffectService icon" width="28" style="vertical-align:-6px" /> SoundEffectService

> 🔄 **Cả hai**

Phát âm thanh hiệu ứng 3D tại vị trí cụ thể trong không gian.

**Lưu ý:**

- `SoundId`: Asset ID âm thanh trên Noobietoria Marketplace.
- `Instance`: tên hoặc GUID của Part/Model; có thể nil khi dùng `Position`.
- `Volume`: number (`0.0` – `1.0`).
- `RolloffDistance`: khoảng cách tắt dần âm thanh (number, đơn vị studs).

```luau
SoundEffectService.Play(SoundId, Position, Volume, RolloffDistance)
             -- Position: {X, Y, Z}
SoundEffectService.PlayOn(SoundId, Instance, Volume, RolloffDistance)
SoundEffectService.Stop(SoundId)
SoundEffectService.SetVolume(SoundId, Volume)
SoundEffectService.Preload(SoundId)     -- Nạp trước asset để giảm độ trễ lần đầu phát
SoundEffectService.Release(SoundId)    -- Giải phóng bộ nhớ khi không còn dùng
```

---

## <img src="/Docs/studio/marketplaceservice.png" alt="MarketplaceService icon" width="28" style="vertical-align:-6px" /> MarketplaceService

> 🖥️ **Server**

Quản lý vật phẩm trên chợ Noobietoria — đăng bán, hạ bán và tra cứu.

**Lưu ý:**

- `Price`: number ≥ 0.
- `Filters`: table tùy chọn để lọc kết quả `Search`, ví dụ `{ CurrencyType = "Coin", MaxPrice = 500 }`.

```luau
MarketplaceService.ListItem(Seller, ItemId, Price, CurrencyType, Quantity)
             -- → ListingId: string
MarketplaceService.DelistItem(ListingId)
MarketplaceService.GetListing(ListingId)       -- → ListingObject | nil
MarketplaceService.GetListings(Seller)         -- → table
MarketplaceService.Search(Query, Filters)      -- → table (ListingObject[])
MarketplaceService.Buy(Username, ListingId, Quantity)
MarketplaceService.GetTransactionHistory(Username, Limit) -- → table
MarketplaceService.OnSale(Seller, Callback)
             -- Callback: function(Seller: string, ListingId: string, Buyer: string)
MarketplaceService.OnPurchase(Username, Callback)
             -- Callback: function(Username: string, ListingId: string)
```

---

## <img src="/Docs/studio/passservice.png" alt="PassService icon" width="28" style="vertical-align:-6px" /> PassService

> 🖥️ **Server** *(alias: GamepassService)*

Kiểm tra và cấp quyền sở hữu Gamepass / đặc quyền cho người chơi.

**Lưu ý:**

- `Permissions`: table quyền hạn tùy chọn đi kèm pass (định dạng tự do).

```luau
PassService.Register(PassId, Name, Description, Permissions)
PassService.Unregister(PassId)
PassService.Grant(Username, PassId)
PassService.Revoke(Username, PassId)
PassService.HasPass(Username, PassId)              -- → bool
PassService.GetPasses(Username)                    -- → table (PassId[])
PassService.GetAll()                               -- → table
PassService.CheckPermission(Username, Permission)  -- → bool
PassService.OnGrant(Username, Callback)
             -- Callback: function(Username: string, PassId: string)
```

---

## <img src="/Docs/studio/tradingservice.png" alt="TradingService icon" width="28" style="vertical-align:-6px" /> TradingService

> 🖥️ **Server**

Giao dịch trực tiếp giữa hai người chơi — vật phẩm hoặc tiền tệ.

**Lưu ý:**

- `TradeId`: được **tự sinh** khi gọi `CreateTrade`.
- `Offer`: `{ items: { { ItemId, Quantity } }, currencies: { { CurrencyType, Amount } } }`.

```luau
TradingService.CreateTrade(FromUsername, ToUsername)    -- → TradeId: string
TradingService.CancelTrade(TradeId)
TradingService.SetOffer(TradeId, Username, Offer)
TradingService.GetOffer(TradeId, Username)              -- → Offer table | nil
TradingService.Confirm(TradeId, Username)               -- Xác nhận phía mình
TradingService.Decline(TradeId, Username)
TradingService.GetTrade(TradeId)                        -- → TradeObject | nil
TradingService.GetTradeHistory(Username, Limit)         -- → table
TradingService.OnTradeComplete(Username, Callback)
             -- Callback: function(Username: string, TradeId: string, Result: "completed" | "cancelled")
```

---

## <img src="/Docs/studio/mailservice.png" alt="MailService icon" width="28" style="vertical-align:-6px" /> MailService

> 🖥️ **Server**

Hòm thư in-game — gửi và nhận tin nhắn kèm vật phẩm đính kèm.

**Lưu ý:**

- `MailId`: được **tự sinh** khi gọi `Send`.
- `Attachments`: `{ items: { { ItemId, Quantity } }, currencies: { { CurrencyType, Amount } } }` (có thể nil).

```luau
MailService.Send(FromUsername, ToUsername, Subject, Body, Attachments)
             -- → MailId: string
MailService.GetInbox(Username, Limit)            -- → table (MailObject[])
MailService.GetOutbox(Username, Limit)           -- → table (MailObject[])
MailService.GetMail(MailId)                      -- → MailObject | nil
MailService.Read(MailId)                         -- Đánh dấu đã đọc
MailService.Delete(MailId)
MailService.ClaimAttachments(MailId, Username)   -- Nhận vật phẩm đính kèm vào inventory
MailService.OnReceive(Username, Callback)
             -- Callback: function(Username: string, MailId: string)
```

---

## <img src="/Docs/studio/craftingservice.png" alt="CraftingService icon" width="28" style="vertical-align:-6px" /> CraftingService

> 🖥️ **Server**

Chế tạo vật phẩm mới từ nguyên liệu trong túi đồ.

**Lưu ý:**

- `Ingredients`: `{ { ItemId: string, Quantity: number } }[]`.
- `Output`: `{ { ItemId: string, Quantity: number, Metadata: table? } }[]`.

```luau
CraftingService.RegisterRecipe(RecipeId, Name, Ingredients, Output)
CraftingService.UnregisterRecipe(RecipeId)
CraftingService.GetRecipe(RecipeId)            -- → RecipeObject | nil
CraftingService.GetAllRecipes()                -- → table
CraftingService.CanCraft(Username, RecipeId)   -- → bool (có đủ nguyên liệu không)
CraftingService.Craft(Username, RecipeId, Quantity)
CraftingService.GetCraftHistory(Username, Limit) -- → table
CraftingService.OnCraft(Username, Callback)
             -- Callback: function(Username: string, RecipeId: string, Quantity: number)
```

---

## <img src="/Docs/studio/inputservice.png" alt="InputService icon" width="28" style="vertical-align:-6px" /> InputService

> 💻 **Client**

Bắt sự kiện đầu vào từ bàn phím, chuột và màn hình cảm ứng.

**Lưu ý:**

- Tất cả hàm `On*` chỉ áp dụng cho người chơi đang dùng `ClientScript` đó.
- `KeyCode`: chuỗi tên phím, ví dụ `"E"`, `"Space"`, `"LeftShift"`, `"F1"`.
- `MouseButton`: `"Left"` | `"Right"` | `"Middle"`.

```luau
InputService.OnKeyPress(Username, KeyCode, Callback)
             -- Callback: function(KeyCode: string)
InputService.OnKeyRelease(Username, KeyCode, Callback)
             -- Callback: function(KeyCode: string)
InputService.OnMouseClick(Username, MouseButton, Callback)
             -- Callback: function(MouseButton: string, Position: {X, Y})
InputService.OnMouseMove(Username, Callback)
             -- Callback: function(Position: {X, Y}, Delta: {X, Y})
InputService.OnTouch(Username, Callback)
             -- Callback: function(Position: {X, Y}, State: "Begin" | "Change" | "End")
InputService.BindAction(ActionName, Keys, Callback)
             -- Keys: string[] (danh sách KeyCode); Callback: function(ActionName: string, State: string)
InputService.UnbindAction(ActionName)
InputService.IsKeyDown(Username, KeyCode)    -- → bool
InputService.Remove(Username, KeyCode)       -- Gỡ tất cả listener cho phím này
```

---

## <img src="/Docs/studio/pathfindingservice.png" alt="PathfindingService icon" width="28" style="vertical-align:-6px" /> PathfindingService

> 🖥️ **Server**

Tìm đường và di chuyển tự động cho NPC trong không gian 3D.

**Lưu ý:** `NpcName` chấp nhận tên hoặc GUID.

```luau
PathfindingService.ComputePath(NpcName, Destination, AgentRadius, AgentHeight)
             -- Destination: {X, Y, Z} hoặc tên Instance; AgentRadius/Height: number
             -- → Waypoints: table | nil
PathfindingService.MoveTo(NpcName, Destination)          -- Di chuyển trực tiếp đến đích
PathfindingService.MoveAlongWaypoints(NpcName, Waypoints) -- Di chuyển theo danh sách điểm trung gian
PathfindingService.Stop(NpcName)
PathfindingService.Pause(NpcName)
PathfindingService.Resume(NpcName)
PathfindingService.GetWaypoints(NpcName)                 -- → table | nil
PathfindingService.OnReached(NpcName, Callback)
             -- Callback: function(NpcName: string)
PathfindingService.OnBlocked(NpcName, Callback)
             -- Callback: function(NpcName: string, BlockedWaypoint: table)
```

---

## <img src="/Docs/studio/proximityservice.png" alt="ProximityService icon" width="28" style="vertical-align:-6px" /> ProximityService

> 🖥️ **Server**

Tạo điểm tương tác (prompt) khi người chơi đến gần đối tượng.

**Lưu ý:** `Instance` chấp nhận tên hoặc GUID.

```luau
ProximityService.Create(PromptId, Instance, Label, KeyCode, MaxDistance, HoldDuration)
             -- KeyCode: chuỗi (mặc định "E"); MaxDistance: number (studs); HoldDuration: number (giây, 0 = nhấn ngay)
ProximityService.Remove(PromptId)
ProximityService.SetVisible(PromptId, Boolean)
ProximityService.SetEnabled(PromptId, Boolean)
ProximityService.GetPrompt(PromptId)           -- → PromptObject | nil
ProximityService.GetAllPrompts()               -- → table
ProximityService.OnTriggered(PromptId, Callback)
             -- Callback: function(Username: string, PromptId: string)
ProximityService.OnEnter(PromptId, Callback)
             -- Callback: function(Username: string, PromptId: string)
ProximityService.OnExit(PromptId, Callback)
             -- Callback: function(Username: string, PromptId: string)
```

---

## <img src="/Docs/studio/localizationservice.png" alt="LocalizationService icon" width="28" style="vertical-align:-6px" /> LocalizationService

> 🔄 **Cả hai**

Đa ngôn ngữ — quản lý và hiển thị nội dung theo ngôn ngữ của người chơi.

**Lưu ý:**

- `Locale`: mã ngôn ngữ BCP 47 (ví dụ: `"vi"`, `"en"`, `"ja"`, `"zh-TW"`).
- `Translations`: `{ [Locale]: string }` — bảng ánh xạ locale → văn bản đã dịch.

```luau
LocalizationService.Register(Key, Translations)
LocalizationService.Unregister(Key)
LocalizationService.Translate(Key, Locale)       -- → string | Key (nếu chưa có bản dịch)
LocalizationService.TranslateFor(Key, Username)  -- Dịch theo ngôn ngữ của người chơi
LocalizationService.GetLocale(Username)          -- → string (Locale)
LocalizationService.SetLocale(Username, Locale)
LocalizationService.GetSupportedLocales()        -- → string[] (danh sách Locale đã đăng ký)
LocalizationService.ImportTable(Translations)    -- Nhập hàng loạt: { [Key]: { [Locale]: string } }
```

---

## <img src="/Docs/studio/physicsservice.png" alt="PhysicsService icon" width="28" style="vertical-align:-6px" /> PhysicsService

> 🖥️ **Server**

Điều khiển trọng lực, lực đẩy và thuộc tính vật lý của đối tượng.

**Lưu ý:** `Instance` chấp nhận tên hoặc GUID.

```luau
PhysicsService.SetGravity(Gravity)              -- Gravity: number (studs/s²; mặc định 196.2)
PhysicsService.GetGravity()                     -- → number
PhysicsService.ApplyForce(Instance, Force, Duration)
             -- Force: {X, Y, Z}; Duration: number (giây; 0 = một frame)
PhysicsService.ApplyImpulse(Instance, Force)    -- Lực tức thời; Force: {X, Y, Z}
PhysicsService.SetVelocity(Instance, Velocity)  -- Velocity: {X, Y, Z}
PhysicsService.GetVelocity(Instance)            -- → {X, Y, Z}
PhysicsService.SetAnchored(Instance, Boolean)   -- Cố định / thả tự do
PhysicsService.SetMass(Instance, Mass)          -- Mass: number (kg)
PhysicsService.SetFriction(Instance, Friction)  -- Friction: number (0.0 – 1.0)
PhysicsService.SetElasticity(Instance, Elasticity) -- Elasticity: number (0.0 – 1.0)
```

---

## <img src="/Docs/studio/textservice.png" alt="TextService icon" width="28" style="vertical-align:-6px" /> TextService

> 🔄 **Cả hai**

Xử lý, đo đạc và định dạng văn bản hiển thị trong game.

**Lưu ý:**

- `Font`: tên font hỗ trợ, ví dụ `"Gotham"`, `"Roboto"`, `"Ubuntu"` (mặc định `"Gotham"`).
- `FontSize`: cỡ chữ (number, mặc định `14`).
- `MaxWidth`: chiều rộng tối đa để xuống dòng (number, đơn vị pixel).
- `Ellipsis`: chuỗi thay thế khi bị cắt ngắn (mặc định `"…"`).
- `MeasureText` trả về `{ Width: number, Height: number }`.
- `WrapText` trả về `string[]` (danh sách dòng sau khi xuống dòng).

```luau
TextService.MeasureText(Text, Font, FontSize)         -- → { Width, Height }
TextService.FitText(Text, MaxWidth, Font, FontSize)   -- → string (cắt vừa MaxWidth)
TextService.WrapText(Text, MaxWidth, Font, FontSize)  -- → string[]
TextService.Truncate(Text, MaxLines, Ellipsis)        -- → string
TextService.Filter(Username, Text)                    -- → string (đã lọc theo cài đặt người chơi)
TextService.EscapeRichText(Text)                      -- → string (escape các ký tự rich text)
TextService.CountCharacters(Text)                     -- → number
```

---

## <img src="/Docs/studio/networkevent.png" alt="NetworkEvent icon" width="28" style="vertical-align:-6px" /> NetworkEvent

> 🔄 **Cả hai** — Instance khai báo trong Explorer

Thiết lập kênh giao tiếp hai chiều giữa Client và Server (tương tự RemoteEvent trong Roblox).

!!! warning "Vị trí đặt"
    `NetworkEvent` phải đặt trong **`ReplicatedStorage`** (hoặc vùng chứa tương đương) để cả `ServerScript` lẫn `ClientScript` cùng nhìn thấy được Instance này. Đặt trong `ServerStorage` thì `ClientScript` không truy cập được.

**Cách triển khai:**

1. `New → NetworkEvent`, đặt tên định danh duy nhất.
2. Server lắng nghe: `NetworkEvent:OnServerEvent(Callback)` — `Callback: function(Username, ...)`
3. Client gửi lên server: `NetworkEvent:FireServer(...)`
4. Server gửi xuống client: `NetworkEvent:FireClient(Username, ...)` hoặc `NetworkEvent:FireAllClients(...)`
5. Client lắng nghe: `NetworkEvent:OnClientEvent(Callback)` — `Callback: function(...)`

**Giới hạn:**

- Payload tối đa: **64 KB** mỗi lần gọi.
- Không gọi `FireAllClients` quá **20 lần/giây** để tránh nghẽn mạng.
- Dữ liệu phải serializable: `string`, `number`, `boolean`, `table` thuần. Không truyền được `function` hay `userdata` — dùng GUID/Name để tham chiếu Instance.

> **Xem thêm:** [`NetworkService`](#networkservice) để điều khiển NetworkEvent bằng Luau; [`NetworkEventService`](#networkeventservice) để giao tiếp không cần Instance.

---

## <img src="/Docs/studio/serverscript.png" alt="ServerScript icon" width="28" style="vertical-align:-6px" /> ServerScript

> 🖥️ **Server** — Instance khai báo trong Explorer

Chứa code Luau chạy hoàn toàn phía server, không được gửi hay thực thi phía client.

!!! note "Quyền truy cập"
    ServerScript có quyền truy cập toàn bộ service phía server: `DataStoreService`, `PlayerService`, `BanService`, `CurrencyService`, … Không thể gọi trực tiếp `CameraService`, `InputService` hay `UIService` — phải giao tiếp qua `NetworkEvent`.

**Cách triển khai:**

- `New → ServerScript`, viết code Luau bên trong.
- Tự động chạy khi game khởi động (server-side).
- Dùng `require()` để nạp `ModuleScript`.

**Lưu ý:**

- Biến và state trong `ServerScript` được chia sẻ giữa tất cả player đang kết nối.
- Lỗi runtime dừng script đó nhưng không làm sập server — ghi vào Server Log.

---

## <img src="/Docs/studio/modulescript.png" alt="ModuleScript icon" width="28" style="vertical-align:-6px" /> ModuleScript

> 🔄 **Cả hai** — Instance khai báo trong Explorer

Chứa code Luau có thể được `require()` từ `ServerScript` hoặc `ClientScript` để tái sử dụng logic.

!!! info "Cache"
    `ModuleScript` chỉ chạy **một lần duy nhất** per môi trường (Server hoặc Client) — kết quả trả về được cache. Lần `require()` tiếp theo nhận lại cùng giá trị mà không chạy lại.

**Cách triển khai:**

- `New → ModuleScript`, `return` một giá trị (thường là table) ở cuối script.
- Dùng `require(ModuleName)` hoặc `InstanceService.Require(Name)` để nạp.

```luau
-- ModuleScript "MathUtils" trong ReplicatedStorage
local MathUtils = {}

function MathUtils.Add(a, b)
    return a + b
end

return MathUtils
```

```luau
-- Dùng trong ServerScript hoặc ClientScript
local MathUtils = InstanceService.Require("MathUtils")
print(MathUtils.Add(3, 5))  -- 8
```

**Lưu ý:**

- `ModuleScript` có thể `require()` `ModuleScript` khác — không được tạo vòng lặp phụ thuộc (circular require).
- Nếu require từ cả Server lẫn Client, script chạy riêng biệt trên mỗi môi trường — không chia sẻ state.
- Tên `ModuleScript` phải duy nhất trong phạm vi game.

---

## <img src="/Docs/studio/clientscript.png" alt="ClientScript icon" width="28" style="vertical-align:-6px" /> ClientScript

> 💻 **Client** — Instance khai báo trong Explorer *(alias: LocalScript)*

Chứa code Luau chạy trên máy của từng người chơi, không có quyền truy cập service phía server.

!!! note "Quyền truy cập"
    `ClientScript` có quyền truy cập: `CameraService`, `UIService`, `InputService`, `EffectService`, `TextService`, … Không thể truy cập `DataStoreService`, `BanService`, hay các service server-only. Mọi tương tác với server phải đi qua `NetworkEvent`.

**Cách triển khai:**

- `New → ClientScript`, thường đặt trong `StarterPlayerScripts`.
- Tự động chạy trên mỗi client khi player tham gia game.
- Có thể `require()` các `ModuleScript` trong `ReplicatedStorage`.

**Lưu ý:**

- Mỗi player có instance `ClientScript` riêng — state không chia sẻ giữa các client.
- Lỗi trên client chỉ ảnh hưởng client đó, không ảnh hưởng server hay client khác.

---

## <img src="/Docs/studio/networkeventservice.png" alt="NetworkEventService icon" width="28" style="vertical-align:-6px" /> NetworkEventService

> 🖥️ **Server** (OnServerEvent, OnServerInvoke, FireClient, FireAllClients, FireAllClientsExcept) / 💻 **Client** (OnClientEvent, FireServer, InvokeServer)

API mạng hoàn toàn bằng code, không cần tạo Instance `NetworkEvent` trước trong Explorer.

**Lưu ý:**

- `EventName`: tên định danh event (chuỗi, phải duy nhất toàn game).
- `Username`: tên người chơi nhận event; `"*"` để gửi tất cả.
- `...`: dữ liệu serializable (string, number, boolean, table thuần).

```luau
-- ===== SERVER SIDE (ServerScript) =====

NetworkEventService.OnServerEvent(EventName, function(Username, ...)
    -- Nhận dữ liệu từ client gửi lên
end)

NetworkEventService.FireClient(EventName, Username, ...)

NetworkEventService.FireAllClients(EventName, ...)

NetworkEventService.FireAllClientsExcept(EventName, ExcludeUsername, ...)

-- Xử lý invoke và trả kết quả về client
NetworkEventService.OnServerInvoke(EventName, function(Username, ...)
    return result   -- Giá trị trả về cho InvokeServer phía client
end)


-- ===== CLIENT SIDE (ClientScript) =====

NetworkEventService.OnClientEvent(EventName, function(...)
    -- Nhận dữ liệu từ server gửi xuống
end)

NetworkEventService.FireServer(EventName, ...)

-- Gọi server và chờ kết quả (blocking)
local result = NetworkEventService.InvokeServer(EventName, ...)


-- ===== CHUNG =====

NetworkEventService.Disconnect(EventName)     -- Ngắt listener
NetworkEventService.Exists(EventName)         -- → bool
```

---

## Ví dụ tổng hợp: Client ↔ Server qua NetworkService + ModuleScript

```luau
-- ModuleScript "EconomyUtils" trong ReplicatedStorage
local EconomyUtils = {}

function EconomyUtils.FormatCoin(amount)
    return "🪙 " .. tostring(amount)
end

return EconomyUtils
```

```luau
-- ServerScript
local EconomyUtils = InstanceService.Require("EconomyUtils")

InstanceService.Create("BuyItem", "NetworkEvent", "ReplicatedStorage")

NetworkService.OnServerEvent("BuyItem", function(Username, ItemId)
    -- Xác thực phía server trước khi thực hiện
    if not InventoryService.HasItem(Username, "Coin_Bundle") then
        NetworkService.FireClient("BuyItem", Username, "error", "Không đủ điều kiện")
        return
    end
    CurrencyService.Deduct(Username, "Coin", 100)
    InventoryService.AddItem(Username, ItemId, 1, nil)
    local balance = CurrencyService.GetBalance(Username, "Coin")
    NetworkService.FireClient("BuyItem", Username, "ok", EconomyUtils.FormatCoin(balance))
end)
```

```luau
-- ClientScript
NetworkService.OnClientEvent("BuyItem", function(status, message)
    if status == "ok" then
        UIService.SetText("CoinLabel", message)
        NotificationService.Send(nil, "Mua thành công", message, "Toast", 3)
    else
        UIService.SetText("ErrorLabel", message)
    end
end)

UIService.OnClick("BuyButton", function()
    NetworkService.FireServer("BuyItem", "Sword_01")
end)
```
