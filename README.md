# NroThucTinh — Unity iOS (Xcode) export

Bản export **Unity → Xcode** của game (bundle id `com.DefaultCompany.DragonBoy002`, tên app `NroThcTnh`, iOS 12.0+, arm64).

## Nội dung

| Thư mục | Vai trò |
| --- | --- |
| `Unity-iPhone.xcodeproj` | Project Xcode gốc do Unity sinh ra (scheme `Unity-iPhone`) |
| `Classes/`, `MainApp/`, `UnityFramework/` | Mã nguồn Objective-C / C++ của Unity runtime |
| `Data/` | Asset của game (`resources.assets`, `resources.assets.resS`, …) |
| `Libraries/` | Thư viện liên kết tĩnh `libiPhone-lib.a`, `libil2cpp.a` |
| `Il2CppOutputProject/` | Mã C++ sinh bởi IL2CPP + toolchain IL2CPP cho macOS |

Các file nhị phân lớn (> 5 MB) được lưu bằng **Git LFS** (`.gitattributes`):
`Libraries/*.a`, `Data/*.assets`, `Data/*.resS`, `Il2CppOutputProject/IL2CPP/build/**/*.dll|*.dylib`.
Clone bắt buộc có LFS:

```bash
git lfs install
git clone https://github.com/dohoanghieu192888-wq/NroThucTinh.git
```

## Build IPA

Máy Windows không build được IPA — cần macOS + Xcode. Có 2 cách:

### 1. Tự build trên macOS

```bash
chmod +x Il2CppOutputProject/IL2CPP/build/deploy_arm64/il2cpp \
         Il2CppOutputProject/IL2CPP/build/deploy_arm64/bee_backend/mac-arm64/bee_backend

xcodebuild -project Unity-iPhone.xcodeproj -scheme Unity-iPhone \
  -configuration Release -sdk iphoneos -destination 'generic/platform=iOS' \
  CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY="" \
  ENABLE_BITCODE=NO ARCHS=arm64 build

mkdir -p stage/Payload
cp -R build/Build/Products/Release-iphoneos/*.app stage/Payload/
(cd stage && zip -qry ../NroThcTnh-unsigned.ipa Payload)
```

### 2. Build tự động bằng GitHub Actions

Workflow [`.github/workflows/build-ipa.yml`](.github/workflows/build-ipa.yml) chạy trên runner `macos-14`:
mỗi lần push lên `main` (hoặc bấm **Run workflow**) sẽ build **IPA chưa ký** và đính kèm trong mục
**Actions → Artifacts** (`NroThcTnh-unsigned-ipa`), kèm `xcodebuild-log` để soi lỗi.

> IPA tạo ra **chưa được ký**, không cài trực tiếp bằng App Store. Cần ký lại bằng Sideloadly,
> AltStore, TrollStore… hoặc ký bằng Apple Developer certificate của bạn.

## Lưu ý

- Git LFS free tier: 1 GB storage / 1 GB bandwidth mỗi tháng — mỗi lần CI chạy sẽ pull lại LFS
  (~350 MB), nên hạn chế chạy workflow liên tục.
- Repo giữ nguyên `project.pbxproj` do Unity sinh; mở bằng Xcode 15+ hoặc mới hơn.