# VaultCLI: Distributed File Storage System 

VaultCLI is a secure, fault-tolerant, and command-line-based distributed file storage system. It uses **AES-256-CBC encryption**, **HMAC-SHA256**, and **Reed-Solomon erasure coding** to ensure data privacy, integrity, and high availability across unreliable or limited storage servers.

---

## Features

- End-to-end encryption using AES-256 in CBC mode with HMAC for integrity  
- File chunking and Reed-Solomon encoding to split data into 10 chunks + 4 parity chunks  
- N+4 Fault tolerance: Any 10 out of 14 chunks are enough to fully recover a file  
- Metadata management for mapping files to chunks, IVs, and HMACs  
- Multithreaded server handling for concurrent client connections  
- Tested with server failures and successful file recovery  

---

## How It Works

### Registration & Authentication
- Users register with a password  
- Passwords are hashed using SHA-256 with a unique salt  

### Upload Process
- File is padded and split into 10 equal chunks  
- Each chunk is encrypted with AES-256-CBC  
- An HMAC is generated per chunk  
- 4 parity chunks are created using Reed-Solomon coding  
- Metadata is stored centrally for decryption and verification  

### Download Process
- System attempts to download any 10 valid chunks  
- HMACs are verified for integrity  
- If original chunks are missing, Reed-Solomon decoding is used  
- Chunks are decrypted and combined into the original file  

### Delete & List
- Users can list all their files  
- Files can be deleted chunk-wise and from metadata  

---

## Project Structure

```
VaultCLI/
├── client.py              # CLI tool for upload/download/delete/list
├── server.py              # Data server logic
├── metadata_server.py     # Central metadata registry
├── reed_solomon.py        # RS encode/decode logic
├── utils.py               # Crypto utils like derive_key, HMAC
├── README.md              # This file
```

---

## Getting Started

### Prerequisites

- Python 3.8+
- `pycryptodome` for AES encryption
- All 14 servers should be running before using the CLI

### Installation

```bash
pip install pycryptodome


### Running Servers

Start the metadata server:

```bash
python metadata_server.py
```

Start 14 data servers:

```bash
for i in {0..13}; do python server.py $i & done
```

---

## CLI Commands

```bash
# Register a new user
python client.py register <user_id> <password>

# Upload a file
python client.py upload <file_path> <user_id> <password>

# List user's files
python client.py list <user_id> <password>

# Download a file
python client.py download <filename> <user_id> <password>

# Delete a file
python client.py delete <filename> <user_id> <password>
```

---

## Example Usage

```bash
python client.py register alice mypassword
python client.py upload ./test.pdf alice mypassword
python client.py list alice mypassword
python client.py download test.pdf alice mypassword
python client.py delete test.pdf alice mypassword
```

---

## Technologies Used

- Python Sockets for TCP-based communication  
- AES-256-CBC from `pycryptodome` for encryption  
- HMAC-SHA256 for integrity verification  
- Reed-Solomon encoding (custom logic or `reedsolo` library)  
- Multi-threading for handling concurrent clients  
- JSON for request and response formatting  

---

## Security Features

- **Password Hashing:** SHA-256 with per-user salt  
- **Key Derivation:** PBKDF2 with 100,000 iterations  
- **Encryption:** AES-256-CBC with random IVs per chunk  
- **Integrity:** HMAC-SHA256 verification per chunk  
- **Authentication:** User credentials are verified before operations  

---

## Architecture

```
Client ←→ Metadata Server (User auth, file metadata)
   ↓
Data Servers (14 total)
├── Servers 0-9:  Data chunks
└── Servers 10-13: Parity chunks
```

---

## Fault Tolerance

The system can handle:

- Up to 4 simultaneous server failures  
- Network partitions or server crashes  
- Partial data corruption (detected via HMAC)  
- Server downtime during upload or download  

Thanks to Reed-Solomon coding, any 10 out of 14 chunks can fully recover the file.

---

## Performance Characteristics

- **Storage Overhead:** ~40% (from 4 parity chunks)  
- **Fault Tolerance:** 4 out of 14 servers can fail  
- **Recovery Time:** Depends on chunk size and network  
- **Scalability:** Linear with number of files and users  

---

## Testing

Tested with:

- Server failures and recoveries  
- File integrity validation after download  
- Concurrent client uploads/downloads  
- Various file sizes (small text → large PDFs)  

---

## License

This project is for academic and learning purposes. You can apply the MIT License if needed.

---

## Acknowledgments

Inspired by cloud resilience principles found in:

- Dropbox (Client-side encryption)  
- IPFS (Distributed file systems)  
- AWS S3 Glacier (Delayed, fault-tolerant retrieval)  
- Tahoe-LAFS (Secure distributed filesystem)  

---

## Contributing

1. Fork the repository  
2. Create a new feature branch  
3. Add your changes  
4. Include tests if possible  
5. Submit a pull request 

---

## Support

For help or issues, please open an issue in the GitHub repository.

---

> Built with for secure, distributed, and resilient file storage.
```
