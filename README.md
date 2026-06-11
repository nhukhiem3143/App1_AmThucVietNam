# 🍜 App 1 – Ẩm Thực Việt Nam

> Ứng dụng giới thiệu các món ăn đặc trưng 3 miền Việt Nam sử dụng dữ liệu chuẩn bị sẵn trong Assets (hoạt động offline hoàn toàn).

---

## 📑 Mục Lục

- [Tổng Quan](#tổng-quan)
- [Bước 1 – Tạo Project Mới](#bước-1--tạo-project-mới)
- [Bước 2 – Cấu Hình build.gradle](#bước-2--cấu-hình-buildgradle)
- [Bước 3 – Tạo Thư Mục Assets & Chuẩn Bị Dữ Liệu](#bước-3--tạo-thư-mục-assets--chuẩn-bị-dữ-liệu)
- [Bước 4 – Khai Báo Resources](#bước-4--khai-báo-resources)
- [Bước 5 – Tạo Data Model](#bước-5--tạo-data-model)
- [Bước 6 – Thiết Kế Layout XML](#bước-6--thiết-kế-layout-xml)
- [Bước 7 – Tạo RecyclerView Adapter](#bước-7--tạo-recyclerview-adapter)
- [Bước 8 – Viết MainActivity](#bước-8--viết-mainactivity)
- [Bước 9 – Tạo DetailActivity](#bước-9--tạo-detailactivity)
- [Bước 10 – Cập Nhật AndroidManifest](#bước-10--cập-nhật-androidmanifest)
- [Bước 11 – Chạy & Kiểm Tra](#bước-11--chạy--kiểm-tra)
- [Hình Ảnh Minh Hoạ](#hình-ảnh-minh-hoạ)
- [Tổng Kết](#tổng-kết)

---

## Tổng Quan

| Mục | Chi tiết |
|---|---|
| **Vấn đề** | Người dùng muốn tra cứu món ăn Việt Nam kể cả khi offline |
| **Giải pháp** | Dữ liệu JSON + ảnh đóng gói sẵn trong Assets |
| **Dữ liệu** | `amthuc.json` – mảng JSON các món ăn theo vùng miền |
| **Thuật toán** | Parse JSON → List → Filter theo vùng |
| **Hiển thị** | RecyclerView + CardView + ImageView từ Assets |
| **Ngôn ngữ** | Java |
| **Min SDK** | API 24 (Android 7.0) |


### 1. Tự Đặt Vấn Đề

**Vấn đề đặt ra:**  
Người học muốn tra cứu các món ăn đặc trưng 3 miền Việt Nam — bao gồm tên, hình ảnh, nguyên liệu và cách nhận biết — kể cả khi **không có kết nối Internet** (đang đi du lịch, vùng sóng yếu, v.v.).

**Giải pháp chọn:**  
Đóng gói toàn bộ dữ liệu (file JSON + ảnh) vào thư mục `assets/` ngay trong app. Khi app được cài lên thiết bị, dữ liệu đi kèm theo — không cần gọi API, không cần mạng.

---

### 2. Mô Tả Đặc Thù Dữ Liệu

Dữ liệu được lưu tại `assets/data/amthuc.json` — là một **mảng JSON phẳng** (flat array), mỗi phần tử đại diện cho một món ăn.

#### 2.1 Cấu trúc một phần tử

```json
{
  "id": 1,
  "ten": "Phở Bò",
  "vung": "Bắc",
  "mo_ta": "Món ăn truyền thống...",
  "nguyen_lieu": ["Xương bò", "Bánh phở", "Hành tây"],
  "image": "images/pho.jpg",
  "dac_trung": "Nước dùng trong, thanh ngọt",
  "do_kho": 3
}
```

#### 2.2 Đặc thù đáng chú ý

| Trường | Kiểu | Đặc thù |
|---|---|---|
| `vung` | String | Giá trị rời rạc: `"Bắc"` / `"Trung"` / `"Nam"` → phù hợp để **phân nhóm, filter** |
| `nguyen_lieu` | **JSONArray lồng** | Mảng bên trong mảng → phải parse 2 cấp, không thể đọc thẳng như String |
| `image` | String (đường dẫn tương đối) | Không phải URL mạng, là đường dẫn trong `assets/` → phải dùng `AssetManager` để mở |
| `do_kho` | int (1–3) | Dữ liệu số nguyên → có thể hiển thị dạng icon 🔥 hoặc sắp xếp theo độ khó |
| `id` | int | Định danh duy nhất → dùng khi cần tra cứu hoặc so sánh chính xác |

**Kết luận đặc thù:**  
Dữ liệu có **trường phân loại rời rạc** (`vung`) và **trường lồng** (`nguyen_lieu`) — đây là 2 điểm cần xử lý đặc biệt, không thể đọc và hiển thị trực tiếp mà không qua bước parse.

---

### 3. Có Cần Tiền Xử Lý Trước Khi Hiển Thị Không?

**Có — cần tiền xử lý ở 2 điểm:**

#### 3.1 Parse JSON → Object Java

Dữ liệu trong Assets là **chuỗi văn bản thuần túy** (raw text). Không thể gán thẳng vào TextView hay ImageView. Phải trải qua pipeline:

```
File JSON (text)
    → đọc bằng InputStream
    → chuyển thành String (UTF-8)
    → parse bằng JSONArray / JSONObject
    → tạo đối tượng MonAn (Java)
    → đưa vào List<MonAn>
    → truyền cho Adapter để hiển thị
```

#### 3.2 Xử lý trường `nguyen_lieu` (mảng lồng)

Trường này là `JSONArray` bên trong `JSONObject` — phải parse vòng lặp riêng:

```java
JSONArray nlArr = obj.getJSONArray("nguyen_lieu");
List<String> nguyenLieu = new ArrayList<>();
for (int j = 0; j < nlArr.length(); j++) {
    nguyenLieu.add(nlArr.getString(j));
}
```

Khi truyền sang `DetailActivity`, danh sách này tiếp tục được **tiền xử lý thêm một lần nữa** — nối thành chuỗi hiển thị với ký tự `•`:

```java
StringBuilder sb = new StringBuilder();
for (String nl : mon.getNguyen_lieu()) {
    sb.append("• ").append(nl).append("\n");
}
intent.putExtra("MON_NGUYEN_LIEU", sb.toString().trim());
```

**Lý do cần bước này:** `TextView` chỉ nhận `String`, không nhận `List<String>` trực tiếp.

#### 3.3 Load ảnh từ Assets

Đường dẫn `"images/pho.jpg"` trong JSON không phải URL — phải mở thủ công qua `AssetManager` và decode thành `Bitmap`:

```java
InputStream is = context.getAssets().open(mon.getImage());
Bitmap bitmap = BitmapFactory.decodeStream(is);
holder.imgMon.setImageBitmap(bitmap);
```

---

### 4. Thuật Toán Xử Lý Dữ Liệu

#### 4.1 Thuật toán Filter theo vùng miền

**Bài toán:** Người dùng nhấn "Miền Bắc" → chỉ hiển thị các món có `vung == "Bắc"`.

**Thuật toán:** Duyệt tuyến tính — **O(n)**

```
Đầu vào : danhSachTatCa (List đầy đủ), vung (String bộ lọc)
Đầu ra  : danhSachHienThi (List đã lọc)

1. Xoá toàn bộ danhSachHienThi
2. Nếu vung == "all":
       Thêm tất cả phần tử từ danhSachTatCa vào danhSachHienThi
   Ngược lại:
       Duyệt từng MonAn trong danhSachTatCa:
           Nếu MonAn.getVung() == vung:
               Thêm MonAn vào danhSachHienThi
3. Gọi adapter.notifyDataSetChanged() để cập nhật UI
```

**Tại sao không cần thuật toán phức tạp hơn?**  
Dữ liệu chỉ có ~6 phần tử (có thể mở rộng đến vài trăm). Với kích thước nhỏ, O(n) là lựa chọn đơn giản, đủ hiệu quả và dễ bảo trì. Nếu dữ liệu lớn hơn (hàng nghìn món), có thể cân nhắc **pre-group** — phân nhóm sẵn thành `Map<String, List<MonAn>>` khi load, tra cứu O(1).

#### 4.2 Thuật toán Parse JSON (đọc một lần khi khởi động)

Chỉ chạy **một lần duy nhất** trong `onCreate` → kết quả lưu vào `danhSachTatCa`. Các lần filter sau đó không đọc lại file — tiết kiệm I/O.

---

### 5. Đối Tượng Hiển Thị Dữ Liệu

#### 5.1 Màn hình danh sách — RecyclerView + CardView

| Lý do chọn RecyclerView | Giải thích |
|---|---|
| **ViewHolder pattern** | Tái sử dụng View đã inflate, không tạo mới mỗi lần scroll → tiết kiệm bộ nhớ, mượt mà |
| **Chỉ render View đang hiển thị** | Không render toàn bộ 100 item cùng lúc → phù hợp danh sách dài |
| **Dễ cập nhật** | `notifyDataSetChanged()` sau filter → UI tự cập nhật |

**CardView** bọc ngoài mỗi item → tạo hiệu ứng nổi (elevation), bo góc, dễ nhìn.

#### 5.2 Màn hình chi tiết — ScrollView + LinearLayout

Nội dung chi tiết (ảnh lớn + văn bản dài) có thể vượt quá chiều cao màn hình → dùng `ScrollView` để cuộn. Bên trong dùng `LinearLayout` (orientation: vertical) để xếp các thành phần từ trên xuống theo thứ tự tự nhiên.

#### 5.3 Sơ đồ luồng hiển thị

```mermaid
flowchart TB
    subgraph DATA["Tầng dữ liệu"]
        direction LR
        A["amthuc.json"] --> B["InputStream"] --> C["JSONArray"] --> D["List&lt;MonAn&gt;"]
    end
    subgraph PROCESS["Tầng xử lý dữ liệu"]
        direction LR
        E["Filter O(n)"] --> F["List&lt;MonAn&gt; đã lọc"] --> G["MonAnAdapter"]
    end
    subgraph UI["Tầng giao diện người dùng"]
        direction LR
        H["RecyclerView"] --> I["CardView<br/>Ảnh / Tên món<br/>Vùng miền / Mô tả"] --> J["Intent + putExtra"] --> K["DetailActivity<br/>ScrollView / Ảnh<br/>Tên / Mô tả / Nguyên liệu"]
    end
    DATA --> PROCESS --> UI
    K -. "Back / finish()" .-> H
```
---

### 6. Lợi Ích Của Cơ Chế Assets

| Lợi ích | Giải thích |
|---|---|
| **Offline hoàn toàn** | Dữ liệu đi kèm app, không phụ thuộc mạng |
| **Tốc độ** | Đọc từ bộ nhớ thiết bị, nhanh hơn HTTP request |
| **Đơn giản** | Không cần server, không cần xử lý lỗi mạng |
| **Phù hợp nội dung tĩnh** | Danh mục món ăn, từ điển, hướng dẫn — ít thay đổi theo thời gian |

**Cú pháp truy cập Assets:**
```java
// Đọc file text/JSON
InputStream is = getAssets().open("data/amthuc.json");

// Đọc file ảnh
InputStream is = getAssets().open("images/pho.jpg");
Bitmap bmp = BitmapFactory.decodeStream(is);
```

**Giới hạn cần lưu ý:** Dữ liệu trong Assets là **chỉ đọc** — không thể ghi/sửa từ code. Nếu cần dữ liệu cập nhật thường xuyên, phải dùng API hoặc database.

---

## Bước 1 – Tạo Project Mới

### 1.1 Mở Android Studio → New Project

```
File → New → New Project
```

### 1.2 Chọn Template

- Chọn **"Empty Views Activity"**
- Click **Next**

<img width="1170" height="828" alt="image" src="https://github.com/user-attachments/assets/57f402b7-39a4-4e9c-8f87-0fa2ebaeb64e" />

### 1.3 Cấu Hình Project

Điền thông tin như sau:

| Trường | Giá trị |
|---|---|
| **Name** | `AmThucVietNam` |
| **Package name** | `com.example.amthucvietnam` |
| **Save location** | Chọn thư mục bạn muốn lưu |
| **Language** | `Java` |
| **Minimum SDK** | `API 24 ("Nougat"; Android 7.0)` |

<img width="1142" height="822" alt="image" src="https://github.com/user-attachments/assets/491dbbdd-d68f-4a52-8b13-f3b88fd45d0e" />

- Click **Finish** → Chờ Android Studio tạo project và sync Gradle lần đầu (~1-3 phút)

<img width="1752" height="978" alt="image" src="https://github.com/user-attachments/assets/1e1af56b-dae2-46de-bc40-7b4f21b80836" />

### 1.4 Cấu Trúc Project Sau Khi Tạo

```
App1_AmThucVietNam/
├── app/
│   ├── src/main/
│   │   ├── java/com/example/amthucvietnam/
│   │   │   └── MainActivity.java       ← File Java tự sinh
│   │   ├── res/
│   │   │   ├── layout/
│   │   │   │   └── activity_main.xml   ← Layout tự sinh
│   │   │   └── values/
│   │   │       ├── strings.xml
│   │   │       ├── colors.xml
│   │   │       └── themes.xml
│   │   └── AndroidManifest.xml
│   └── build.gradle                    ← Gradle cấp app
└── build.gradle                        ← Gradle cấp project
```
<img width="1620" height="923" alt="image" src="https://github.com/user-attachments/assets/0e203d24-a630-4820-b503-593339ce0a37" />


---

## Bước 2 – Cấu Hình build.gradle

Mở file `app/build.gradle.kts` và kiểm tra / thêm:

```gradle
android {
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.amthucvietnam"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }

    buildTypes {
        getByName("release") {
            isMinifyEnabled = false
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation("androidx.appcompat:appcompat:1.6.1")
    implementation("com.google.android.material:material:1.11.0")
    implementation("androidx.constraintlayout:constraintlayout:2.1.4")

    // RecyclerView – hiển thị danh sách
    implementation("androidx.recyclerview:recyclerview:1.3.2")
    // CardView – hiệu ứng card cho từng item
    implementation("androidx.cardview:cardview:1.0.0")
}
```

Sau khi sửa → Click **"Sync Now"** (thanh vàng xuất hiện phía trên)

<img width="1724" height="985" alt="image" src="https://github.com/user-attachments/assets/e93843bc-1643-4a49-b391-e2e4356222e3" />

---

## Bước 3 – Tạo Thư Mục Assets & Chuẩn Bị Dữ Liệu

### 3.1 Tạo thư mục Assets trong Android Studio

```
Chuột phải vào thư mục main → New → Directory
→ Gõ: assets
→ Enter
```
<img width="1647" height="959" alt="image" src="https://github.com/user-attachments/assets/b2b9b374-025f-44bb-b5d7-ed00edf90731" />


### 3.2 Tạo cấu trúc thư mục con


Tạo thủ công 2 thư mục con:
```
assets/
├── data/        ← chứa file JSON
└── images/      ← chứa ảnh các món ăn
```

<img width="1636" height="969" alt="image" src="https://github.com/user-attachments/assets/e638973f-552e-498d-a1fa-b18371952205" />

### 3.3 Tạo file `amthuc.json`

Trong thư mục `assets/data/`, tạo file `amthuc.json`:

```json
[
  {
    "id": 1,
    "ten": "Phở Bò",
    "vung": "Bắc",
    "mo_ta": "Món ăn truyền thống của người Hà Nội, nổi tiếng với nước dùng trong và ngọt từ xương bò hầm nhiều giờ cùng các gia vị hồi, quế, gừng.",
    "nguyen_lieu": ["Xương bò", "Bánh phở", "Thịt bò tái", "Hành tây", "Gừng nướng", "Quế", "Hồi", "Rau thơm"],
    "image": "images/pho.jpg",
    "dac_trung": "Nước dùng trong, thanh ngọt, thơm mùi hồi quế",
    "do_kho": 3
  },
  {
    "id": 2,
    "ten": "Bún Chả",
    "vung": "Bắc",
    "mo_ta": "Bún chả Hà Nội gồm bún tươi, chả viên và chả miếng nướng than hoa, chấm nước mắm pha chua ngọt.",
    "nguyen_lieu": ["Bún tươi", "Thịt nạc vai", "Thịt ba chỉ", "Nước mắm", "Tỏi", "Ớt", "Chanh", "Rau sống"],
    "image": "images/buncha.jpg",
    "dac_trung": "Chả nướng than hoa, nước chấm chua ngọt đặc trưng",
    "do_kho": 2
  },
  {
    "id": 3,
    "ten": "Bún Bò Huế",
    "vung": "Trung",
    "mo_ta": "Bún bò Huế nổi tiếng với nước dùng đậm đà, cay nồng từ sả và mắm ruốc, ăn kèm giò heo và huyết.",
    "nguyen_lieu": ["Bún tươi", "Thịt bò", "Giò heo", "Sả", "Mắm ruốc", "Ớt", "Hành tím", "Rau sống"],
    "image": "images/bunbohue.jpg",
    "dac_trung": "Nước dùng đỏ cay nồng, thơm sả mắm ruốc",
    "do_kho": 2
  },
  {
    "id": 4,
    "ten": "Mì Quảng",
    "vung": "Trung",
    "mo_ta": "Mì Quảng đặc trưng với sợi mì vàng rộng bản, ít nước dùng sền sệt, ăn kèm tôm thịt và bánh tráng nướng.",
    "nguyen_lieu": ["Mì Quảng", "Tôm", "Thịt heo", "Trứng cút", "Đậu phộng", "Bánh tráng nướng", "Rau sống"],
    "image": "images/miquang.jpg",
    "dac_trung": "Ít nước, sợi mì vàng, ăn với bánh tráng nướng giòn",
    "do_kho": 2
  },
  {
    "id": 5,
    "ten": "Bánh Mì Sài Gòn",
    "vung": "Nam",
    "mo_ta": "Bánh mì Sài Gòn nổi tiếng toàn thế giới với vỏ bánh giòn tan, nhân đa dạng gồm pate, chả lụa, thịt nguội, dưa chua và rau thơm.",
    "nguyen_lieu": ["Bánh mì", "Pate", "Chả lụa", "Thịt nguội", "Dưa chua", "Dưa leo", "Rau mùi", "Tương ớt"],
    "image": "images/banhmi.jpg",
    "dac_trung": "Vỏ giòn tan, nhân phong phú, ăn nhanh tiện lợi",
    "do_kho": 1
  },
  {
    "id": 6,
    "ten": "Hủ Tiếu Nam Vang",
    "vung": "Nam",
    "mo_ta": "Hủ tiếu Nam Vang có nước dùng trong ngọt từ xương heo, ăn kèm thịt nạc, tôm, gan heo và các loại rau giá.",
    "nguyen_lieu": ["Hủ tiếu", "Xương heo", "Tôm tươi", "Thịt nạc", "Gan heo", "Giá đỗ", "Hành lá", "Tỏi phi"],
    "image": "images/hutieu.jpg",
    "dac_trung": "Nước dùng trong, ngọt thanh, ăn khô hoặc nước đều được",
    "do_kho": 2
  }
]
```

<img width="1737" height="957" alt="image" src="https://github.com/user-attachments/assets/b4490fc6-2333-4737-a689-2831df044542" />

### 3.4 Copy ảnh vào thư mục `assets/images/`

- Tìm hoặc tải ảnh các món ăn (JPG/PNG)
- Đổi tên ảnh đúng với trường `"image"` trong JSON: `pho.jpg`, `buncha.jpg`, `bunbohue.jpg`, `miquang.jpg`, `banhmi.jpg`, `hutieu.jpg`
- Copy vào `assets/images/` bằng Windows Explorer

<img width="1274" height="586" alt="image" src="https://github.com/user-attachments/assets/caaa75b5-5f97-426b-9f40-1c247dd6a7c7" />


---

## Bước 4 – Khai Báo Resources

### 4.1 `res/values/strings.xml`

```xml
<resources>
    <string name="app_name">Ẩm Thực Việt Nam</string>
    <string name="all_regions">Tất Cả</string>
    <string name="north">Miền Bắc</string>
    <string name="central">Miền Trung</string>
    <string name="south">Miền Nam</string>
    <string name="ingredients_title">Nguyên liệu</string>
    <string name="specialty_title">Đặc trưng</string>
    <string name="region_label">Vùng miền</string>
    <string name="error_load_data">Lỗi tải dữ liệu!</string>
    <string name="no_result">Không có món ăn nào.</string>
</resources>
```
<img width="1734" height="957" alt="image" src="https://github.com/user-attachments/assets/52274562-4a28-440f-93a3-e67a1bbea4ff" />

### 4.2 `res/values/colors.xml`

```xml
<resources>
    <color name="primary">#C62828</color>
    <color name="primary_dark">#8E0000</color>
    <color name="accent">#FF8F00</color>
    <color name="background">#FFF8F0</color>
    <color name="card_background">#FFFFFF</color>
    <color name="text_primary">#212121</color>
    <color name="text_secondary">#757575</color>
    <color name="badge_bac">#1565C0</color>
    <color name="badge_trung">#2E7D32</color>
    <color name="badge_nam">#E65100</color>
    <color name="divider">#EEEEEE</color>
</resources>
```
<img width="1680" height="894" alt="image" src="https://github.com/user-attachments/assets/2570ca02-fa39-4dc8-8050-ff445fe48c4d" />

### 4.3 `res/values/dimens.xml`

```xml
<resources>
    <dimen name="title_text_size">20sp</dimen>
    <dimen name="body_text_size">14sp</dimen>
    <dimen name="caption_text_size">12sp</dimen>
    <dimen name="card_margin">8dp</dimen>
    <dimen name="card_padding">12dp</dimen>
    <dimen name="image_height">180dp</dimen>
    <dimen name="card_radius">12dp</dimen>
</resources>
```
<img width="1736" height="946" alt="image" src="https://github.com/user-attachments/assets/cd8c1343-81ab-43dd-9028-4b0163c6f1df" />

---

## Bước 5 – Tạo Data Model

### 5.1 Tạo package `model`

```
Chuột phải vào package gốc (com.example.amthucvietnam)
→ New → Package
→ Gõ: model
→ Enter
```
<img width="1550" height="988" alt="image" src="https://github.com/user-attachments/assets/ef0f1cb9-06de-4d43-9736-7ac7c4432c7a" />

### 5.2 Tạo class `MonAn.java`

```
Chuột phải vào package model
→ New → Java Class
→ Gõ: MonAn
→ Enter
```
<img width="1539" height="959" alt="image" src="https://github.com/user-attachments/assets/b8b306f4-67b1-48f2-8070-881d533dadc0" />


Nội dung file:

```java
package com.example.amthucvietnam.model;

import java.util.List;

public class MonAn {
    private int id;
    private String ten;
    private String vung;
    private String mo_ta;
    private List<String> nguyen_lieu;
    private String image;
    private String dac_trung;
    private int do_kho;

    public MonAn(int id, String ten, String vung, String mo_ta,
                 List<String> nguyen_lieu, String image,
                 String dac_trung, int do_kho) {
        this.id = id;
        this.ten = ten;
        this.vung = vung;
        this.mo_ta = mo_ta;
        this.nguyen_lieu = nguyen_lieu;
        this.image = image;
        this.dac_trung = dac_trung;
        this.do_kho = do_kho;
    }

    public int getId()                   { return id; }
    public String getTen()               { return ten; }
    public String getVung()              { return vung; }
    public String getMo_ta()             { return mo_ta; }
    public List<String> getNguyen_lieu() { return nguyen_lieu; }
    public String getImage()             { return image; }
    public String getDac_trung()         { return dac_trung; }
    public int getDo_kho()               { return do_kho; }
}
```

---

## Bước 6 – Thiết Kế Layout XML

### 6.1 Layout item RecyclerView – `res/layout/item_mon_an.xml`

```
Chuột phải vào res/layout → New → Layout Resource File
→ File name: item_mon_an
→ Root element: androidx.cardview.widget.CardView
→ OK
```
<img width="1327" height="917" alt="image" src="https://github.com/user-attachments/assets/76da135f-6e59-4cf0-9d5f-d6ac569d154e" />

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="@dimen/card_margin"
    app:cardCornerRadius="@dimen/card_radius"
    app:cardElevation="4dp"
    app:cardBackgroundColor="@color/card_background">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <!-- Ảnh món ăn -->
        <ImageView
            android:id="@+id/imgMon"
            android:layout_width="match_parent"
            android:layout_height="@dimen/image_height"
            android:scaleType="centerCrop"
            android:contentDescription="@string/app_name" />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="@dimen/card_padding">

            <!-- Dòng tên + badge vùng -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:gravity="center_vertical">

                <TextView
                    android:id="@+id/tvTen"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:textSize="@dimen/title_text_size"
                    android:textStyle="bold"
                    android:textColor="@color/text_primary" />

                <TextView
                    android:id="@+id/tvVung"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:paddingHorizontal="8dp"
                    android:paddingVertical="3dp"
                    android:textSize="@dimen/caption_text_size"
                    android:textColor="#FFFFFF"
                    android:background="@drawable/badge_background"
                    android:textStyle="bold" />

            </LinearLayout>

            <!-- Mô tả -->
            <TextView
                android:id="@+id/tvMoTa"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="6dp"
                android:textSize="@dimen/body_text_size"
                android:textColor="@color/text_secondary"
                android:maxLines="2"
                android:ellipsize="end" />

        </LinearLayout>
    </LinearLayout>
</androidx.cardview.widget.CardView>
```
<img width="1690" height="969" alt="image" src="https://github.com/user-attachments/assets/248177e3-7c52-49a9-89c3-b7080e00ba3c" />

### 6.2 Tạo drawable badge – `res/drawable/badge_background.xml`

```
Chuột phải vào res/drawable → New → Drawable Resource File
→ File name: badge_background
→ OK
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <solid android:color="@color/primary" />
    <corners android:radius="20dp" />
</shape>
```
<img width="1569" height="905" alt="image" src="https://github.com/user-attachments/assets/0572f720-64b2-4b44-9191-d4514fdeec8a" />


### 6.3 Layout màn hình chính – `res/layout/activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@color/background">

    <!-- Thanh tiêu đề -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/app_name"
        android:textSize="22sp"
        android:textStyle="bold"
        android:textColor="@color/primary"
        android:padding="16dp"
        android:background="@color/card_background"
        android:elevation="4dp" />

    <!-- Thanh filter vùng miền -->
    <HorizontalScrollView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:scrollbars="none"
        android:background="@color/card_background">

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:padding="8dp"
            android:gravity="center_vertical">

            <Button
                android:id="@+id/btnTatCa"
                style="@style/Widget.MaterialComponents.Button.OutlinedButton"
                android:layout_width="wrap_content"
                android:layout_height="36dp"
                android:layout_marginEnd="6dp"
                android:text="@string/all_regions"
                android:textSize="12sp" />

            <Button
                android:id="@+id/btnBac"
                style="@style/Widget.MaterialComponents.Button.OutlinedButton"
                android:layout_width="wrap_content"
                android:layout_height="36dp"
                android:layout_marginEnd="6dp"
                android:text="@string/north"
                android:textSize="12sp" />

            <Button
                android:id="@+id/btnTrung"
                style="@style/Widget.MaterialComponents.Button.OutlinedButton"
                android:layout_width="wrap_content"
                android:layout_height="36dp"
                android:layout_marginEnd="6dp"
                android:text="@string/central"
                android:textSize="12sp" />

            <Button
                android:id="@+id/btnNam"
                style="@style/Widget.MaterialComponents.Button.OutlinedButton"
                android:layout_width="wrap_content"
                android:layout_height="36dp"
                android:text="@string/south"
                android:textSize="12sp" />

        </LinearLayout>
    </HorizontalScrollView>

    <!-- Danh sách món ăn -->
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:clipToPadding="false"
        android:padding="4dp" />

</LinearLayout>
```
<img width="1624" height="953" alt="image" src="https://github.com/user-attachments/assets/eacc05d0-603c-4701-a0b5-b303e96e2fc6" />

### 6.4 Layout màn hình chi tiết – `res/layout/activity_detail.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/background">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <!-- Ảnh lớn -->
        <ImageView
            android:id="@+id/imgDetail"
            android:layout_width="match_parent"
            android:layout_height="250dp"
            android:scaleType="centerCrop" />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp">

            <!-- Tên + Badge vùng -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:gravity="center_vertical">

                <TextView
                    android:id="@+id/tvDetailTen"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:textSize="24sp"
                    android:textStyle="bold"
                    android:textColor="@color/text_primary" />

                <TextView
                    android:id="@+id/tvDetailVung"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:paddingHorizontal="12dp"
                    android:paddingVertical="4dp"
                    android:textColor="#FFFFFF"
                    android:background="@drawable/badge_background"
                    android:textStyle="bold" />
            </LinearLayout>

            <!-- Đặc trưng -->
            <TextView
                android:id="@+id/tvDetailDacTrung"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="8dp"
                android:textSize="14sp"
                android:textColor="@color/primary"
                android:textStyle="italic" />

            <!-- Mô tả -->
            <TextView
                android:id="@+id/tvDetailMoTa"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="12dp"
                android:textSize="15sp"
                android:textColor="@color/text_primary"
                android:lineSpacingMultiplier="1.5" />

            <!-- Tiêu đề nguyên liệu -->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="16dp"
                android:text="@string/ingredients_title"
                android:textSize="16sp"
                android:textStyle="bold"
                android:textColor="@color/primary" />

            <!-- Danh sách nguyên liệu -->
            <TextView
                android:id="@+id/tvDetailNguyenLieu"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="6dp"
                android:textSize="14sp"
                android:textColor="@color/text_secondary"
                android:lineSpacingMultiplier="1.8" />

        </LinearLayout>
    </LinearLayout>
</ScrollView>
```
<img width="1620" height="962" alt="image" src="https://github.com/user-attachments/assets/0cedecb1-3629-4a65-ba82-8d786f3dcadc" />

---

## Bước 7 – Tạo RecyclerView Adapter

### 7.1 Tạo package `adapter`

```
Chuột phải vào package gốc → New → Package → adapter
```

### 7.2 Tạo class `MonAnAdapter.java`

```
Chuột phải vào package adapter → New → Java Class → MonAnAdapter
```

```java
package com.example.amthucvietnam.adapter;

import android.content.Context;
import android.content.Intent;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;
import com.example.amthucvietnam.DetailActivity;
import com.example.amthucvietnam.R;
import com.example.amthucvietnam.model.MonAn;
import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class MonAnAdapter extends RecyclerView.Adapter<MonAnAdapter.ViewHolder> {

    private final Context context;
    private final List<MonAn> danhSach;

    public MonAnAdapter(Context context, List<MonAn> danhSach) {
        this.context = context;
        this.danhSach = danhSach;
    }

    @NonNull
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(context)
                .inflate(R.layout.item_mon_an, parent, false);
        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        MonAn mon = danhSach.get(position);

        // Gán text từ data – KHÔNG hardcode
        holder.tvTen.setText(mon.getTen());
        holder.tvVung.setText(mon.getVung());
        holder.tvMoTa.setText(mon.getMo_ta());

        // Đổi màu badge theo vùng
        int badgeColor;
        switch (mon.getVung()) {
            case "Bắc":  badgeColor = context.getColor(R.color.badge_bac);   break;
            case "Trung": badgeColor = context.getColor(R.color.badge_trung); break;
            default:      badgeColor = context.getColor(R.color.badge_nam);   break;
        }
        holder.tvVung.setBackgroundTintList(
            android.content.res.ColorStateList.valueOf(badgeColor)
        );

        // Load ảnh từ Assets
        try {
            InputStream is = context.getAssets().open(mon.getImage());
            Bitmap bitmap = BitmapFactory.decodeStream(is);
            holder.imgMon.setImageBitmap(bitmap);
            is.close();
        } catch (IOException e) {
            holder.imgMon.setImageResource(android.R.drawable.ic_menu_gallery);
        }

        // Xử lý sự kiện click → mở DetailActivity
        holder.itemView.setOnClickListener(v -> {
            Intent intent = new Intent(context, DetailActivity.class);
            intent.putExtra("MON_TEN",      mon.getTen());
            intent.putExtra("MON_VUNG",     mon.getVung());
            intent.putExtra("MON_MO_TA",    mon.getMo_ta());
            intent.putExtra("MON_IMAGE",    mon.getImage());
            intent.putExtra("MON_DAC_TRUNG",mon.getDac_trung());

            // Truyền danh sách nguyên liệu thành chuỗi
            StringBuilder sb = new StringBuilder();
            for (String nl : mon.getNguyen_lieu()) {
                sb.append("• ").append(nl).append("\n");
            }
            intent.putExtra("MON_NGUYEN_LIEU", sb.toString().trim());

            context.startActivity(intent);
        });
    }

    @Override
    public int getItemCount() {
        return danhSach.size();
    }

    static class ViewHolder extends RecyclerView.ViewHolder {
        ImageView imgMon;
        TextView tvTen, tvVung, tvMoTa;

        ViewHolder(View itemView) {
            super(itemView);
            imgMon  = itemView.findViewById(R.id.imgMon);
            tvTen   = itemView.findViewById(R.id.tvTen);
            tvVung  = itemView.findViewById(R.id.tvVung);
            tvMoTa  = itemView.findViewById(R.id.tvMoTa);
        }
    }
}
```
<img width="1724" height="967" alt="image" src="https://github.com/user-attachments/assets/5aed2791-cdcb-49fd-a7a6-9ac7e2ff14a8" />

---

## Bước 8 – Viết MainActivity

Mở file `MainActivity.java`:

```java
package com.example.amthucvietnam;

import android.os.Bundle;
import android.widget.Button;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;
import com.example.amthucvietnam.adapter.MonAnAdapter;
import com.example.amthucvietnam.model.MonAn;
import org.json.JSONArray;
import org.json.JSONObject;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    private RecyclerView recyclerView;
    private MonAnAdapter adapter;
    private final List<MonAn> danhSachTatCa  = new ArrayList<>();
    private final List<MonAn> danhSachHienThi = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 1. Ánh xạ View
        recyclerView = findViewById(R.id.recyclerView);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));

        // 2. Load dữ liệu từ Assets
        loadDataFromAssets();

        // 3. Khởi tạo Adapter
        adapter = new MonAnAdapter(this, danhSachHienThi);
        recyclerView.setAdapter(adapter);

        // 4. Gắn filter buttons
        setupFilterButtons();
    }

    /**
     * Đọc file amthuc.json từ Assets, parse thành List<MonAn>
     */
    private void loadDataFromAssets() {
        try {
            // Mở file từ Assets
            InputStream inputStream = getAssets().open("data/amthuc.json");
            int size = inputStream.available();
            byte[] buffer = new byte[size];
            inputStream.read(buffer);
            inputStream.close();
            String jsonString = new String(buffer, "UTF-8");

            // Parse JSONArray
            JSONArray jsonArray = new JSONArray(jsonString);
            for (int i = 0; i < jsonArray.length(); i++) {
                JSONObject obj = jsonArray.getJSONObject(i);

                // Parse mảng nguyên liệu
                JSONArray nlArr = obj.getJSONArray("nguyen_lieu");
                List<String> nguyenLieu = new ArrayList<>();
                for (int j = 0; j < nlArr.length(); j++) {
                    nguyenLieu.add(nlArr.getString(j));
                }

                danhSachTatCa.add(new MonAn(
                    obj.getInt("id"),
                    obj.getString("ten"),
                    obj.getString("vung"),
                    obj.getString("mo_ta"),
                    nguyenLieu,
                    obj.getString("image"),
                    obj.getString("dac_trung"),
                    obj.getInt("do_kho")
                ));
            }

            // Hiển thị toàn bộ ban đầu
            danhSachHienThi.addAll(danhSachTatCa);

        } catch (IOException e) {
            Toast.makeText(this, getString(R.string.error_load_data),
                Toast.LENGTH_SHORT).show();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * Lọc danh sách theo vùng miền
     */
    private void filterByVung(String vung) {
        danhSachHienThi.clear();
        if (vung.equals("all")) {
            danhSachHienThi.addAll(danhSachTatCa);
        } else {
            for (MonAn mon : danhSachTatCa) {
                if (mon.getVung().equals(vung)) {
                    danhSachHienThi.add(mon);
                }
            }
        }
        adapter.notifyDataSetChanged();
    }

    private void setupFilterButtons() {
        Button btnTatCa = findViewById(R.id.btnTatCa);
        Button btnBac   = findViewById(R.id.btnBac);
        Button btnTrung = findViewById(R.id.btnTrung);
        Button btnNam   = findViewById(R.id.btnNam);

        btnTatCa.setOnClickListener(v -> filterByVung("all"));
        btnBac.setOnClickListener(v   -> filterByVung("Bắc"));
        btnTrung.setOnClickListener(v -> filterByVung("Trung"));
        btnNam.setOnClickListener(v   -> filterByVung("Nam"));
    }
}
```
<img width="1750" height="966" alt="image" src="https://github.com/user-attachments/assets/be726fda-1fb4-4e47-a4cc-0d04988a4b1d" />


---

## Bước 9 – Tạo DetailActivity

### 9.1 Tạo Activity mới

```
Chuột phải vào package gốc → New → Activity → Empty Views Activity
→ Activity Name: DetailActivity
→ Layout Name: activity_detail  (tự điền)
→ Finish
```

<img width="1142" height="826" alt="image" src="https://github.com/user-attachments/assets/9a716b50-b3e3-455e-8d6d-092afa3595de" />

> **Lưu ý:** Android Studio tự động thêm `<activity android:name=".DetailActivity"/>` vào `AndroidManifest.xml`

### 9.2 Viết `DetailActivity.java`

```java
package com.example.amthucvietnam;

import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.Bundle;
import android.widget.ImageView;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import java.io.IOException;
import java.io.InputStream;

public class DetailActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_detail);

        // Bật nút Back trên ActionBar
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        }

        // Nhận dữ liệu từ Intent
        String ten        = getIntent().getStringExtra("MON_TEN");
        String vung       = getIntent().getStringExtra("MON_VUNG");
        String moTa       = getIntent().getStringExtra("MON_MO_TA");
        String imagePath  = getIntent().getStringExtra("MON_IMAGE");
        String dacTrung   = getIntent().getStringExtra("MON_DAC_TRUNG");
        String nguyenLieu = getIntent().getStringExtra("MON_NGUYEN_LIEU");

        // Ánh xạ và gán View
        setTitle(ten); // Tiêu đề ActionBar

        TextView tvTen        = findViewById(R.id.tvDetailTen);
        TextView tvVung       = findViewById(R.id.tvDetailVung);
        TextView tvMoTa       = findViewById(R.id.tvDetailMoTa);
        TextView tvDacTrung   = findViewById(R.id.tvDetailDacTrung);
        TextView tvNguyenLieu = findViewById(R.id.tvDetailNguyenLieu);
        ImageView imgDetail   = findViewById(R.id.imgDetail);

        // Gán text – lấy từ Intent, không hardcode
        tvTen.setText(ten);
        tvVung.setText(vung);
        tvMoTa.setText(moTa);
        tvDacTrung.setText(dacTrung);
        tvNguyenLieu.setText(nguyenLieu);

        // Load ảnh từ Assets
        try {
            InputStream is = getAssets().open(imagePath);
            Bitmap bitmap  = BitmapFactory.decodeStream(is);
            imgDetail.setImageBitmap(bitmap);
            is.close();
        } catch (IOException e) {
            imgDetail.setImageResource(android.R.drawable.ic_menu_gallery);
        }
    }

    // Xử lý nút Back trên ActionBar
    @Override
    public boolean onSupportNavigateUp() {
        finish();
        return true;
    }
}
```
<img width="1751" height="975" alt="image" src="https://github.com/user-attachments/assets/088b763c-0ac8-4c05-8334-ccb6f916207c" />

---

## Bước 10 – Cập Nhật AndroidManifest

Mở `AndroidManifest.xml` và kiểm tra:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.amthucvietnam">

    <!-- App này KHÔNG cần Internet vì dùng Assets offline -->

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.AmThucVietNam">

        <!-- MainActivity là màn hình khởi động -->
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- DetailActivity – Android Studio tự thêm khi tạo Activity mới -->
        <activity
            android:name=".DetailActivity"
            android:exported="false"
            android:parentActivityName=".MainActivity" />

    </application>
</manifest>
```
<img width="1450" height="965" alt="image" src="https://github.com/user-attachments/assets/b891623b-38e0-4826-85c7-6fc2a95fc41c" />

---

## Bước 11 – Chạy & Kiểm Tra

### 11.1 Kết nối thiết bị hoặc khởi động Emulator

**Dùng thiết bị thật:**
```
1. Thiết bị: Cài đặt → Giới thiệu điện thoại → Nhấn 7 lần vào "Số hiệu bản dựng"
2. Thiết bị: Bật "Tuỳ chọn lập trình viên" → Bật "Gỡ lỗi USB"
3. Cắm cáp USB vào máy tính
4. Chọn "Cho phép gỡ lỗi USB" trên thiết bị
```

### 11.2 Build & Run

```
Click nút ▶ (Run) trên toolbar
Hoặc: Shift + F10
```

<img width="441" height="975" alt="image" src="https://github.com/user-attachments/assets/4aa7ceaf-a1d2-4fdc-bba8-4cd4adebbbe6" />


### 11.3 Checklist kiểm tra

```
☐ App mở được, hiển thị danh sách món ăn
☐ Ảnh hiển thị đúng (load từ Assets)
☐ Tên, vùng miền, mô tả hiển thị đúng
☐ Nút filter "Miền Bắc" → chỉ hiện món Bắc
☐ Nút filter "Tất Cả" → hiện lại tất cả
☐ Click vào món → mở màn hình chi tiết
☐ Chi tiết hiển thị đầy đủ: ảnh, tên, đặc trưng, mô tả, nguyên liệu
☐ Nút Back trên DetailActivity hoạt động
☐ App chạy được khi TẮT WIFI (test offline)
```
<img width="441" height="975" alt="image" src="https://github.com/user-attachments/assets/85750cb1-6cd4-461d-9b9d-f3055fb4722e" />
<img width="444" height="969" alt="image" src="https://github.com/user-attachments/assets/7cf745a4-1730-4c55-a121-68870a90b4df" />
<img width="426" height="971" alt="image" src="https://github.com/user-attachments/assets/498e5e16-f1e7-4b08-a26e-8b1dafd0809e" />
<img width="441" height="968" alt="image" src="https://github.com/user-attachments/assets/169a1c20-0b8c-4dbf-ac26-0cc7f0de77c0" />


---


## Tổng Kết

### Những gì đã làm được

| Kỹ thuật | Áp dụng |
|---|---|
| **Assets** | Đọc JSON + ảnh không cần Internet |
| **JSON Parsing** | `JSONArray`, `JSONObject` → `List<MonAn>` |
| **RecyclerView + Adapter** | Hiển thị danh sách dạng card có ảnh |
| **Intent + Bundle** | Truyền dữ liệu giữa 2 Activity |
| **Resources** | `@string`, `@color`, `@dimen` – không hardcode |
| **Filter** | Lọc danh sách theo trường `vung` |
| **Offline** | 100% hoạt động không cần mạng |

### Đặc thù dữ liệu & Thuật toán

```
Dữ liệu: Mảng JSON phẳng, mỗi phần tử có field phân loại "vung"
Thuật toán filter: O(n) – duyệt toàn bộ List, giữ phần tử thoả điều kiện
Hiển thị: RecyclerView với ViewHolder pattern – tái sử dụng View, tiết kiệm bộ nhớ
Ảnh: Decode bitmap từ InputStream của AssetManager
```
