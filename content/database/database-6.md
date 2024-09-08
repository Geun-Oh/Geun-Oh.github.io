+++
title = 'Database 6'
date = 2024-09-08T22:54:19+09:00
draft = false
+++

### 데이터베이스 시스템 구현

세부 구현을 시작한다.

먼저, 마운트된 공간에 데이터를 적재하고, 해당 데이터를 조회하는 기본적인 기능을 수행하도록 Table 구조체를 작성한다.

예시 코드를 작성할 때에는 마운트 포인트를 `pwd + /data` 로 임의 설정했다.
내부에 테이블 종류에 따라 폴더를 추가 구성하고, 해당 폴더 내부에 `.schema` 파일을 생성하여 schema 구조를 정의하고, `.csv` 확장자로 데이터를 적재했다.
데이터 저장 형태는 여러 형태가 권장되고 있었는데, 별도의 예시를 찾기 어려워 테이블 구조를 저장하고 관리하기 좋은 csv 형태를 선택했다.

아래는 구현된 Table 구조체 코드이다.

```rust
// table.rs
use std::collections::BTreeMap;
use std::fs::{self, File};
use std::io::{BufRead, BufReader, BufWriter, Write};
use std::path::{Path, PathBuf};
use std::error::Error;
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
struct TableSchema {
	columns: Vec<String>,
}

pub struct Table {
	rows: Vec<Vec<String>>,
	index: BTreeMap<String, usize>,
	schema: TableSchema,
	path: PathBuf,
}


impl Table {
	pub fn new(table_name: &str, schema: Vec<String>) -> Result<Table, Box<dyn Error>> {
		let table_dir = format!("./data/{}", table_name);
		let schema_file = format!("{}/{}.schema", table_dir, table_name);
		let csv_file = format!("{}/{}.csv", table_dir, table_name);
		let path = PathBuf::from(table_dir);

		if !Path::new(&path).exists() {
			fs::create_dir_all(&path)?;
		}

		let schema = if Path::new(&schema_file).exists() {
			Table::load_schema(&schema_file)?
		} else {
			TableSchema { columns: schema }
		};

		let rows = if Path::new(&csv_file).exists() {
			Table::load_csv(&csv_file)?
		} else {
			Vec::new()
		};

		Ok(Table {
			rows,
			index: BTreeMap::new(),
			schema,
			path: PathBuf::from(path),
		})
	}

	pub fn load_schema(schema_path: &str) -> Result<TableSchema, Box<dyn Error>> {
		let file = File::open(schema_path)?;
		let reader = BufReader::new(file);
		let schema: TableSchema = serde_json::from_reader(reader)?;
		Ok(schema)
	}

	pub fn load_csv(csv_path: &str) -> Result <Vec<Vec<String>>, Box<dyn Error>> {
		let file = File::open(csv_path)?;
		let reader = BufReader::new(file);
		let mut rows = Vec::new();

		for line in reader.lines() {
			let line = line?;
			let row: Vec<String> = line.split(',').map(|s| s.to_string()).collect();
			rows.push(row);
		}

		Ok(rows)
	}

	pub fn save_schema(&self) -> Result<(), Box<dyn Error>> {
		let schema_file = format!("{}/{}.schema", self.path.display(), "table");
		println!("Saving Schema to: {}", schema_file);
		let file = File::create(schema_file)?;
		let writer = BufWriter::new(file);
		serde_json::to_writer(writer, &self.schema)?;
		Ok(())
	}

	pub fn save_csv(&self) -> Result<(), Box<dyn Error>> {
		let csv_file = format!("{}/{}.csv", self.path.display(), "table");
		println!("Saving CSV to: {}", csv_file);
		let file = File::create(csv_file)?;
		let mut writer = BufWriter::new(file);

		for row in &self.rows {
			writeln!(writer, "{}", row.join(","))?;
		}

		writer.flush()?;
		Ok(())
	}

	pub fn insert(&mut self, row: Vec<String>) -> Result<(), Box<dyn Error>> {
		let key = row[0].clone(); // use first row as a key
		self.index.insert(key.clone(), self.rows.len());
		self.rows.push(row);

		self.save_csv();
		Ok(())
	}

	pub fn select(&self, key: &String) -> Option<&Vec<String>> {
		self.index.get(key).map(|&i| &self.rows[i])
	}
}

```

큰 설명이 필요한 코드가 존재하지는 않고, 단순히 파일 경로와 파일 이름을 확인하여 I/O 를 수행하는 테이블이다.

### Transparent Database Encryption

.csv 파일 단위로 암호화를 실행한다. 이후 해당 파일을 읽어들일 때 복호화를 진행한다.

```rust
// encryption.rs
use aes::cipher::{BlockDecrypt, KeyInit};
use aes::Aes256;
use aes::cipher::{BlockEncrypt, generic_array::GenericArray};
use rand::Rng;
use base64::{engine, Engine};
use std::error::Error;
use std::fs::{File};
use std::io::{Read, Write};

const KEY_SIZE: usize = 32;

pub fn generate_key() -> [u8; KEY_SIZE] {
    let mut key = [0u8; KEY_SIZE];
    rand::thread_rng().fill(&mut key[..]);
    key
}

fn encrypt_data(key: &[u8], data: &[u8]) -> Result<String, Box<dyn Error>> {
    let cipher = Aes256::new(&GenericArray::from_slice(key));
    let mut buffer = data.to_vec();
    let padding = 16 - (buffer.len() % 16);
    buffer.extend(vec![padding as u8; padding]);

    for chunk in buffer.chunks_mut(16) {
        cipher.encrypt_block(&mut GenericArray::from_mut_slice(chunk));
    }
    
    Ok(engine::general_purpose::STANDARD.encode(&buffer))
}

fn decrypt_data(key: &[u8], encode_data: &str) -> Result<Vec<u8>, Box<dyn Error>> {
    let cipher = Aes256::new(&GenericArray::from_slice(key));
    let data = engine::general_purpose::STANDARD.decode(encode_data)?;

    let mut buffer = data;
    for chunk in buffer.chunks_mut(16) {
        cipher.decrypt_block(&mut GenericArray::from_mut_slice(chunk));
    }

    let padding = buffer.last().unwrap_or(&0) & 0xFF;
    let end = buffer.len() - padding as usize;

    Ok(buffer[..end].to_vec())
}

pub fn encrypt_file(file_path: &str, key: &[u8]) -> Result<(), Box<dyn Error>> {
    let mut file = File::open(file_path)?;
    let mut data = Vec::new();
    file.read_to_end(&mut data)?;

    let encrypted_data = encrypt_data(key, &data)?;
    let mut file = File::create(file_path)?;
    file.write_all(encrypted_data.as_bytes())?;
    Ok(())
}

pub fn decrypt_file(file_path: &str, key: &[u8]) -> Result<(), Box<dyn Error>> {
    let mut file = File::open(file_path)?;
    let mut encoded_data = String::new();
    file.read_to_string(&mut encoded_data)?;

    let decrypted_data = decrypt_data(key, &encoded_data)?;
    let mut file = File::create(file_path)?;
    file.write_all(&decrypted_data)?;
    Ok(())
}
```

위의 구현된 내용처럼 AES 기반 암호화를 진행한다.

실제 기본 파일은 다음과 같은 형태인데,

```csv
1,Alice,25
2,Bob,30
```

다음과 같이 암호화된 형태로 적재된다.

```csv
VBodlpqBIeCPbBgBbTVGBckgTmw+Vbq6amR7l8heUS4=
```

이후 다시 조회하는 경우 decryption을 진행하여 데이터를 불러온다.