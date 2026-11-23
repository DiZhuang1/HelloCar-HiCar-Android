# 第三方组件许可声明

本项目包含或链接以下第三方开源组件。各组件的著作权归其原作者所有，本项目仅按各自许可证的条款使用。

| 组件 | 目录 | 许可证 | 用途 | 说明 |
| --- | --- | --- | --- | --- |
| DeviceAuth (deviceauth_lite) | `third_party/deviceauth` | Apache-2.0 | HiChain 认证协议栈（PAKE、挑战应答、密钥协商） | OpenHarmony / HiCar 设备侧开源实现，非从设备提取 |
| HUKS (huks_lite) | `third_party/huks` | Apache-2.0 | 设备密钥库（密钥生成/导入/签名/派生） | 同上 |
| securec | `third_party/securec` | Apache-2.0 | 安全字符串/内存函数 | 同上 |
| cJSON | `third_party/cjson` | MIT | 认证信封与能力表的 JSON 编解码 | 未修改 |
| mbedTLS | `third_party/mbedtls` | Apache-2.0 | 密码学原语（仅编译 crypto 部分） | 未修改 |
| OpenSSL | `third_party/openssl` | Apache-2.0 | `arm64-v8a/libcrypto.a` 预编译库 | 随包提供，便于离线构建 |
| Rust crates | `rust-core/Cargo.toml` | 见各 crate | `serde`/`serde_json`(MIT OR Apache-2.0)、`libloading`(MIT)、`libc`(MIT OR Apache-2.0)、`jni`(MIT OR Apache-2.0) | 由 Cargo 解析 |

## 分发注意

- 若二次分发本项目的 APK，请同时保留 `LICENSE` 与本文件，并保留 `third_party/` 下各组件的 `LICENSE` 原文。
- Apache-2.0 组件要求保留版权声明与 NOTICE 文件（如组件内附带）。
- 本项目自身的代码采用 Apache-2.0，与上述组件兼容；**不提供任何形式的商业授权或担保**。
- 本项目**不包含**任何从商用设备中提取的二进制或反编译产物：`libhellocar_hichain.so` 由 `build_auth_lib.ps1` 用 NDK 从上述开源源码编译得到，可复现。
