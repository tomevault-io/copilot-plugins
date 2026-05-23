## cursorrule

> Bộ quy tắc phát triển ứng dụng React Native + Expo cho dự án "Bác sĩ Lúa".

@description
Bộ quy tắc phát triển ứng dụng React Native + Expo cho dự án "Bác sĩ Lúa". 
Mục tiêu: đảm bảo project có cấu trúc chuẩn (/src), hiệu năng tối ưu trên Android, quản lý i18n chuẩn, quyền runtime rõ ràng, flow chụp ảnh + watermark thông qua backend, và backend Node.js + MongoDB nhẹ, ổn định, có Swagger và cơ chế wake-on-sleep cho Render. 
@end

@context
Ngôn ngữ: TypeScript (ưu tiên) / JavaScript (hàng phụ)
Framework: Expo Managed Workflow (create-expo-app)
Mục tiêu triển khai: Android-first tối ưu, tương thích iOS, code có thể scale, dễ maintain, CI-friendly, documentation luôn cập nhật.
@end

@rules
{
  "always": {
    "description": "Những quy tắc bắt buộc toàn cục.",
    "rules": [
       "Trước khi CHẠY CODE: luôn hỏi user xác nhận rõ ràng những câu hỏi bạn chưa hiểu (ví dụ: 'Tôi cần làm rõ vấn đề abc này và tôi gợi ý dùng phương án này, bạn có đồng ý không?').",
       "Trước khi XOÁ hoặc CHỈNH SỬA dữ liệu: luôn hỏi xác nhận với user trước khi thực hiện, và hiển thị tóm tắt dữ liệu sẽ bị ảnh hưởng nếu có thể.",
       "SAU MỖI LẦN user gửi yêu cầu: trước khi thực hiện hành động chính được yêu cầu, luôn xác nhận lại rằng hành động đó đúng với ý định của user.",
      "Luôn dùng TypeScript cho business logic; file UI có thể là .tsx. Nếu dùng JS, phải có JSDoc cho mọi function export.",
      "Project root phải có file `.env-example` chứa tất cả biến môi trường dùng trong app và backend (ví dụ: API_URL, GOOGLE_CLIENT_ID, FACEBOOK_APP_ID, SENTRY_DSN nếu dùng). Không commit `.env` thật vào VCS.",
      "Không để `console.log`, `debugger` trong bản build production; dùng logger controllable (ví dụ `debug` hoặc Sentry) và tắt theo biến môi trường.",
      "Mọi API network call phải thông qua module riêng `/src/services/api.ts` và sử dụng axios instance có timeout, retry logic (3 lần), và cancel token support."
    ]
  },

  "projectStructure": {
    "description": "Cấu trúc Hybrid với Expo Router + /src cho business logic.",
    "rules": [
      "Root chứa: /app (Expo Router screens), /src (business logic), và các file cấu hình (package.json, app.json, tsconfig.json, .eslintrc, .prettierrc, .env-example)",
      "Cấu trúc Hybrid:",
      "  /app - Expo Router screens (file-based routing): (tabs), modal, _layout.tsx. Screens chỉ chứa UI layout và import logic từ /src",
      "  /src/components - các component tái sử dụng (Header, Button, Modal, Avatar, WatermarkPreview, SkeletonLoader, etc.)",
      "  /src/hooks - custom hooks (useAuth, usePermissions, useI18n, useCameraFlow, useFetch)",
      "  /src/services - api clients, auth service, storage service",
      "  /src/constants - colors, spacing, fonts, keys",
      "  /src/i18n - cấu hình i18n và files locales (vi.json, en.json)",
      "  /src/utils - helper functions, formatters, validators",
      "  /src/types - TypeScript types và interfaces global",
      "  /src/assets - hình, icon, fonts (tối ưu kích thước & webp nếu phù hợp)",
      "Import xuyên folder phải dùng path aliases (ví dụ: @/components, @/hooks, @/services) được cấu hình trong tsconfig.json. Không dùng nhiều `../../..`.",
      "Screens trong /app KHÔNG chứa business logic - chỉ import hooks/components từ /src và render UI."
    ]
  },

  "appJsonAndBuild": {
    "description": "Cấu hình app.json để tên hiển thị là \"Bác sĩ Lúa\" và build tối ưu.",
    "rules": [
      "app.json / app.config.js phải đặt `name` và `displayName` phù hợp: displayName = \"Bác sĩ Lúa\".",
      "Sử dụng `expo-asset` và `expo-font` để preload assets và fonts trước khi show root screen.",
      "Dùng `eas build` cho production; cấu hình eas.json để có profile 'production' và 'development'.",
      "Prioritize minimal bundle: only install packages cần thiết, tránh native modules không hỗ trợ Expo Managed Workflow.",
      "Kiểm tra và test `expo start --clear` trước khi build release.",
      "Định nghĩa App version và buildNumber trong app.json và cập nhật mỗi release."
    ]
  },

  "authAndAccounts": {
    "description": "Đăng nhập/Đăng ký qua email, số điện thoại, Google, Facebook.",
    "rules": [
      "Xây module auth tại `/src/services/auth` xử lý: login, register, socialLogin, refreshToken, logout.",
      "Email/password và phone/SMS OTP đều phải xác thực trên backend. Trên app: validate form trước khi gửi (email regex, phone format).",
      "Social login (Google, Facebook) sử dụng `expo-auth-session` hoặc SDK tương thích Expo; token exchange phải thực hiện an toàn qua backend (backend verify token với Google/Facebook và trả JWT của hệ thống).",
      "Không lưu mật khẩu ở client. Lưu token/refreshToken trong SecureStore (Expo SecureStore) hoặc encrypted storage.",
      "Thêm flow must_change_password, forgot password và verify phone flow (OTP)."
    ]
  },

  "permissionsAndPrivacy": {
    "description": "Quy tắc xử lý quyền runtime và privacy.",
    "rules": [
      "Khi app lần đầu chạy, show permission rationale UI rõ ràng trước khi gọi requestPermissions (camera, location).",
      "Đề xuất quyền: CAMERA và LOCATION (coarse/fine) — chỉ request khi user cần tính năng tương ứng (không request cùng lúc nếu chưa dùng).",
      "Dùng `expo-permissions` / `expo-location` / `expo-camera` để kiểm tra & request quyền.",
      "Luôn show fallback UI (explain how to enable in settings) nếu user từ chối quyền.",
      "Tất cả quyền cần phải được khai báo rõ ràng trong Privacy Policy (link trong app).",
      "Không request quyền background location hoặc camera chạy nền trừ khi có lý do chính đáng và mô tả rõ trong store listing."
    ]
  },

  "cameraAndWatermarkFlow": {
    "description": "Flow mở camera, chụp ảnh, gửi backend để gán watermark dựa trên định vị, lưu ảnh.",
    "rules": [
      "Tạo hook `useCameraFlow` chịu trách nhiệm: kiểm tra quyền, mở camera, capture, preview, send to API.",
      "Ảnh chỉ được lưu local sau khi backend trả về ảnh đã gán watermark (backend sẽ trả URL hoặc base64).",
      "Tất cả ảnh upload phải có metadata kèm theo: { userId, timestamp, lat, lng, deviceId, orientation }. Metadata phải được tạo bằng location lấy ngay trước khi chụp (hoặc sau chụp trước khi gửi).",
      "Không gắn watermark local — watermark phải được xử lý bởi backend để đảm bảo nhất quán (theo yêu cầu).",
      "Kích thước ảnh trước khi upload: resize/crop phía client để tiết kiệm băng thông (ví dụ max width 1280) nhưng giữ đủ chất lượng cho watermark. Dùng thư viện như `expo-image-manipulator`.",
      "Upload là multipart/form-data và backend cần xác thực user token (Bearer).",
      "Trên màn hình preview show progress upload và lên skeleton trong khi chờ backend trả ảnh đã watermark.",
      "Nếu backend trả lỗi — show retry với exponential backoff (max 3 attempts) và log lỗi ở service layer."
    ]
  },

  "i18n": {
    "description": "Thiết lập i18n (Tiếng Việt mặc định) và luôn cập nhật khi code thay đổi.",
    "rules": [
      "Dùng `i18next` + `react-i18next` + `expo-localization`.",
      "Locale files nằm ở `/src/i18n/locales/vi.json` và `/src/i18n/locales/en.json`.",
      "Default language = 'vi'. Hỗ trợ chuyển đổi trong settings và lưu `appLanguage` vào AsyncStorage.",
      "Tất cả text render phải thông qua `t('key')`. Không hardcode string.",
      "Khi thêm/xóa key i18n — cập nhật `AppLogicConfig.Md` và chạy script kiểm tra (node script) để phát hiện key missing (CI job).",
      "Khi deploy code mới, CI pipeline phải chạy `i18n-check` script để đảm bảo không mất key dịch — nếu mất key, build fail.",
      "Thực hiện fallbackLng = 'vi' và logging cảnh báo khi thiếu key trong dev."
    ]
  },

  "skeleton": {
    "description": "Skeleton UI cho loading mượt mà.",
    "rules": [
      "Skeleton components nằm trong `/src/components/skeletons`. Mỗi screen có thể có Skeleton component riêng (ví dụ: HomeScreenSkeleton, CameraScreenSkeleton).",
      "Dùng thư viện `moti` hoặc `react-native-reanimated` để tạo shimmer/fade nhẹ cho skeleton animation.",
      "Skeleton chỉ hiển thị khi `isLoading === true`. Không gọi API khi đang show skeleton (data fetch trước/sau quản lý ở screen).",
      "Skeleton components phải memoized (React.memo/useMemo).",
      "Không để skeleton tồn tại quá lâu — nếu > 3s hiển thị retry hoặc error placeholder.",
      "Không lồng skeleton trong FlatList — dùng SkeletonItem cho từng phần tử."
    ]
  },

  "performance": {
    "description": "Tối ưu tốc độ load app và runtime.",
    "rules": [
      "Preload fonts và critical assets trước khi mount AppRoot (show minimal splash).",
      "Tách bundle bằng dynamic imports cho các screens hiếm dùng (React.lazy / dynamic import).",
      "Sử dụng `expo-updates` để cung cấp patch nhẹ (chỉ khi tuân thủ policy).",
      "Sử dụng `expo-image` hoặc `react-native-fast-image` (nếu sử dụng bare) để tối ưu hình ảnh.",
      "Giảm số lượng re-render: dùng useCallback, useMemo, React.memo, keyExtractor trên FlatList.",
      "Giới hạn số permission requests và heavy native modules khởi tạo lúc cold start."
    ]
  },

  "backendRequirements": {
    "description": "Yêu cầu backend (Node.js + MongoDB) và cách deploy trên Render.",
    "rules": [
      "Backend dùng Node.js (TS recommended) + Express (hoặc Fastify cho performance) + Mongoose cho MongoDB.",
      "Cơ sở dữ liệu: MongoDB Atlas (hoặc self-host); collections tối thiểu: `users`, `photos`, `sessions`.",
      "Collection `users` chứa: { _id, email, phone, passwordHash, displayName, socialIds, roles, createdAt, updatedAt }.",
      "Collection `photos` chứa: { _id, userId, originalUrl, watermarkedUrl, metadata: { lat, lng, timestamp, device }, status, createdAt }.",
      "Endpoints cơ bản: /auth/register, /auth/login, /auth/social, /auth/refresh, /photos/upload, /photos/{id}, /photos/list, /health, /swagger.json.",
      "Sử dụng JWT cho auth, refresh token lưu an toàn (db). HTTPS bắt buộc (Render cung cấp SSL).",
      "Build API phải expose Swagger UI (ví dụ `/api/docs` hoặc `/swagger`) để quản lý và test endpoints.",
      "Tạo file `.env-example` chứa biến: PORT, MONGO_URI, JWT_SECRET, JWT_EXPIRES, RENDER_INTERNAL_URL (nếu cần), GOOGLE_CLIENT_ID, FACEBOOK_APP_SECRET, CRON_SECRET.",
      "Vì Render có thể sleep — triển khai cron/ping bằng `node-cron` hoặc `cron` để ping internal health endpoint mỗi 2–3 phút. Tức: cron job server tự ping hoặc sử dụng external uptime service; phải bảo vệ bằng CRON_SECRET để tránh abuse.",
      "Tối ưu nhẹ: bật gzip compression, set cache headers cho assets tĩnh, stream uploads nếu cần."
    ]
  },

  "renderAndCronWake": {
    "description": "Cách chữa Render sleep.",
    "rules": [
      "Trong app backend, tạo route `/health` trả 200 simple JSON.",
      "Cấu hình cron job trong backend (node-cron) để mỗi 2 phút thực hiện fetch GET `/health` nếu Render account không cho external uptime. (Lưu ý: ping nội bộ này hữu dụng nhưng không nên lạm dụng; nếu dùng external uptime service — an toàn hơn).",
      "Bảo vệ cron ping bằng token (CRON_SECRET) hoặc chỉ ping từ internal worker process.",
      "Log cron wake attempts nhẹ để debug, nhưng không spam logs."
    ]
  },

  "swaggerAndDocs": {
    "description": "API documentation & dev UX.",
    "rules": [
      "Bao gồm Swagger (OpenAPI 3) auto-generated từ code (ví dụ swagger-jsdoc hoặc tsoa) và hiển thị UI tại `/api/docs`.",
      "Mọi endpoint mới phải được document trong Swagger; CI job kiểm tra coverage của docs (ví dụ kiểm tra nếu route không có OpenAPI doc → fail build).",
      "Update `BackendConfig.Md` mỗi khi thay đổi schema, endpoint, hoặc authentication flow."
    ]
  },

  "ciAndReleaseFlow": {
    "description": "CI, pre-deploy checks và release checklist.",
    "rules": [
      "CI pipeline (GitHub Actions/GitLab CI) phải chạy: lint, typecheck, i18n-check, unit tests, expo-doctor (for app), node tests (for backend).",
      "Pre-release checklist: permissions, privacy policy link, icon & splash, app.json, version bump, BackendConfig.Md & AppLogicConfig.Md updated.",
      "Build artifact size nên bị cảnh báo nếu vượt threshold (configurable)."
    ]
  },

  "docsAndUpdating": {
    "description": "Quy tắc cập nhật hai file docs auto/manual.",
    "rules": [
      "AppLogicConfig.Md phải mô tả: cấu trúc /app và /src, Expo Router file-based routing, screen list trong /app, i18n keys quan trọng, camera+watermark flow, permission flow, path aliases, và environment variables used in app.",
      "BackendConfig.Md phải mô tả: DB schemas (users, photos), endpoint list, auth flow, error codes, env vars, deployment notes (render + cron), swagger url, and sample requests/responses.",
      "Mỗi Pull Request lớn phải kèm diff update cho các file doc tương ứng (reviewers kiểm tra doc trước merge).",
      "Đặt tag [DOCS] trong commit message khi chỉnh docs."
    ]
  },

  "expoRouter": {
    "description": "Quy tắc sử dụng Expo Router (file-based routing).",
    "rules": [
      "Sử dụng Expo Router cho navigation - folder /app là entry point chính.",
      "File-based routing: /app/index.tsx (home), /app/(tabs)/_layout.tsx (tab navigator), /app/modal.tsx (modal screen).",
      "Dùng expo-router hooks: useRouter(), useLocalSearchParams(), usePathname() cho navigation.",
      "Screens trong /app chỉ chứa UI structure và import business logic từ /src/hooks và /src/services.",
      "Layouts (_layout.tsx) quản lý navigation structure (Stack, Tabs) và global providers.",
      "Typed routes được enable qua experiments.typedRoutes trong app.json."
    ]
  },

  "security": {
    "description": "Bảo mật dữ liệu và best practices.",
    "rules": [
      "Tất cả mật khẩu phải hash trên backend (bcrypt/argon2).",
      "Không gửi secrets trong logs hoặc client. Sentry/monitoring phải tắt chi tiết stack traces ở prod.",
      "Thực hiện rate limiting cho endpoints nhạy cảm (login, OTP).",
      "Dùng HTTPS cho mọi gọi API; implement CORS whitelist cho web clients nếu cần.",
      "Thực hiện validation và sanitization cho mọi input (use celebrate/Joi hoặc Zod)."
    ]
  }
}
@end

@example
📘 Ví dụ `app.json` với Expo Router:
```json
{
  "expo": {
    "name": "DoctorRice",
    "slug": "doctorrice",
    "displayName": "Bác sĩ Lúa",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./src/assets/icon.png",
    "splash": {
      "image": "./src/assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "android": {
      "package": "com.yourcompany.doctorrice",
      "adaptiveIcon": {
        "foregroundImage": "./src/assets/icon.png",
        "backgroundColor": "#ffffff"
      }
    },
    "plugins": ["expo-router"],
    "experiments": {
      "typedRoutes": true
    }
  }
}

---
> Source: [NguyenDaiPhong/DoctorRice-main](https://github.com/NguyenDaiPhong/DoctorRice-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
