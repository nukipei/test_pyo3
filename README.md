# Rye × Pyo3 (maturin)のサンプルコード

## 前提

- Windows 11で実行
- rustup, ryeがインストール済み

## 手順

### 1. プロジェクトの作成

今回は`test_pyo3`配下にプロジェクトを作成する。

```bash
cd test_pyo3
rye init
```

### 2. パッケージの追加

```bash
rye add --dev maturin
```

### 3. パッケージのインストール

```bash
rye sync
```
### 4. Cargo.tomlの作成

```toml
[package]
name = "test_pyo3"
version = "0.1.0"
edition = "2021"

[lib]
path = "rust_ext/lib.rs"

[dependencies]
pyo3 = { version = "0.21.2", features = ["extension-module"] }
```

### 5. Rustサンプルコードの作成

以下を、`rust_ext/lib.rs`に記述。

```rust:
use pyo3::prelude::*;

/// Formats the sum of two numbers as string.
#[pyfunction]
fn sum_as_string(a: usize, b: usize) -> PyResult<String> {
    Ok((a + b).to_string())
}

/// A Python module implemented in Rust. The name of this function must match
/// the `lib.name` setting in the `Cargo.toml`, else Python will not be able to
/// import the module.
#[pymodule]
fn string_sum(m: &Bound<'_, PyModule>) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(sum_as_string, m)?)?;
    Ok(())
}
```

引用: https://github.com/PyO3/pyo3

### 6. Rustバイナリ呼び出し用のPythonコードの記述

以下を、`string_sum/__init__.py`に記述。

```python
from .string_sum import *
```

### 7. ビルド

以下を実行。成功すれば、`string_sum`配下に`string_sum.cp3xx-win_amd64.pyd`が生成される。

```bash
maturin develop
```

### 8. Pythonコードの実行

以下を、`src/test_pyo3.py`に記述し、実行。
```python
import string_sum
string_sum.sum_as_string(1, 2)
```

## トラブルシューティング

### 1. `maturin develop`実行時に、`Caused by: The module name must not contain a minus `-` ` が発生

pyproject.tomlの `project.name`に`-`が含まれると発生。（例："test-pyo3"）
このプロジェクト、pyproject.tomlにでは以下を追加し、解決。
```toml
[tool.maturin]
module-name = "string_sum"
```

### 2. `maturin develop`実行時に、`Caused by: pip install finished with "exit code: 1"` が発生

ryeが作る仮想環境.venvにはpipが含まれていないため、maturinのビルド時にエラーが発生する。手動でpipを追加し、解決。
```bash
rye add --dev pip
```

引用: https://rye-up.com/guide/rust/#iterating

## 参考

- https://github.com/PyO3/maturin
- https://github.com/PyO3/pyo3
- https://www.maturin.rs/
- https://zenn.dev/yyu/articles/3b87c9499fddde
- https://zenn.dev/kitchy/articles/5b4578ca9a87d9