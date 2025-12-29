# ParseDMARC-go

A **Go implementation** of the DMARC report parser, based on the original Python [parsedmarc](https://github.com/domainaware/parsedmarc) project.

## 📋 Conversion & Enhancements

The conversion to Go was done with **Claude AI**, adding significant improvements:

✅ **Core enhancements:**
- ClickHouse storage with pre-configured Grafana dashboard
- HTTP reporting method (RUA/RUF with https/http scheme URI)  
- Prometheus daemon mode monitoring (IMAP + HTTP)
- Directory-based output mode

❌ **Not converted** (due to lack of testing capability):
- Elasticsearch/Opensearch/Splunk storage
- Microsoft Graph and Gmail API support

## 🌟 Core Features

### 📊 **Report Parsing** - Industry leading format support
- ✅ **DMARC Aggregate Reports** ([RFC 7489](https://datatracker.ietf.org/doc/html/rfc7489))
  - Draft and 1.0 standard formats
  - Compressed file support (GZIP, ZIP)
  - Enhanced error reporting with line numbers

- ✅ **Forensic/Failure Reports** ([RFC 6591 ARF](https://datatracker.ietf.org/doc/html/rfc6591)) 
  - Plain text format parsing
  - **🆕 MIME multipart email parsing** (LinkedIn, Domain.de, Netease)
  - **🆕 Base64-encoded attachment support**
  - Automatic format detection and fallback

- ✅ **SMTP TLS Reports** ([RFC 8460](https://datatracker.ietf.org/doc/html/rfc8460))
  - Direct JSON format parsing
  - **🆕 MIME email format parsing** (Google, other providers)
  - **🆕 Base64 + GZIP compressed attachment pipeline** (`application/tlsrpt+gzip`)
  - Legacy compressed file support (GZIP, ZIP)

### 🌐 **Data Enhancement**
- ✅ IP address geolocation (MaxMind database integration)
- ✅ Reverse DNS resolution with caching
- ✅ Base domain extraction and normalization
- ✅ Enhanced error diagnostics with precise line numbers

### 📡 **Multiple Input Methods**
- ✅ **IMAP Email Processing** - Monitor mailboxes for incoming reports
  - TLS/SSL connection support
  - Automatic email archiving/deletion
  - Configurable check intervals
  
- ✅ **HTTP API Server** - Receive reports via HTTP POST/PUT ([IETF draft](https://datatracker.ietf.org/doc/html/draft-kucherawy-dmarc-base-02#appendix-B.6))
  - Rate limiting and request validation
  - Multiple content-type support (`application/xml`, `application/json`, `message/rfc822`)
  - File upload size limits and security

### 💾 **Flexible Output & Storage**
- ✅ **JSON and CSV output formats** with configurable fields
- ✅ **Multiple output modes:**
  - **File mode**: Concatenate all reports in single file
  - **🆕 Directory mode**: Save each report as separate timestamped file  
  - **Stdout**: Direct console output for piping
- ✅ **ClickHouse database storage** with optimized schema
- ✅ **Email delivery** via SMTP with attachment support
- ✅ **Kafka streaming** for real-time processing pipelines

### 📈 **Production Monitoring**
- ✅ **Built-in Prometheus metrics** for observability
- ✅ **Health check endpoints** for load balancer integration
- ✅ **Structured logging** with configurable levels (JSON/console)
- ✅ **Performance metrics** (parsing duration, success/failure rates)


## Quick Start

```bash
# Download and install
curl -L -o parsedmarc-go https://github.com/CaboneMailServer/ParseDMarc-go/releases/latest/download/parsedmarc-go-linux-amd64
chmod +x parsedmarc-go

# Parse a report
./parsedmarc-go -input report.xml

# Run as daemon
./parsedmarc-go -daemon -config config.yaml
```

For detailed usage instructions, see the **[📖 Documentation](#-documentation)** below.

## 🗄️ ClickHouse Database Schema

The program automatically creates **optimized production-ready tables** with proper indexing, partitioning, and performance optimizations:

### 📋 **dmarc_aggregate_reports**
**Main aggregate report metadata table**
- Report metadata (organization, report ID, date range)
- Policy information (DMARC alignment settings)
- Monthly partitioning by report date
- Bloom filter indexes on org_name and report_id

### 📊 **dmarc_aggregate_records** 
**Individual aggregate report records**
- Source IP analysis (IP, country, reverse DNS)
- Authentication results (SPF, DKIM, DMARC alignment)
- Message counts and policy evaluation results
- Monthly partitioning with geolocation indexing

### 🔍 **dmarc_forensic_reports**
**Forensic/failure report details**
- Authentication failure analysis
- Source information and sample headers
- Parsed sample message content
- Indexed by arrival date and source IP

### 🔐 **dmarc_smtp_tls_reports** 
**SMTP TLS report metadata** 
- Organization and policy information
- Success/failure session counts
- Policy domain and type information
- Time-based partitioning for performance

### ⚠️ **dmarc_smtp_tls_failures** 
**Detailed SMTP TLS failure analysis**
- Failure types and error codes
- MTA connection details (sending/receiving IPs)
- MX hostname and HELO information
- Normalized for efficient failure pattern analysis

### 🚀 **Performance Features**
- **Time-based partitioning**: Monthly partitions for optimal query performance
- **Bloom filter indexes**: Fast lookups on report IDs and domains
- **Optimized data types**: Efficient storage with proper nullable fields
- **Query-optimized structure**: Denormalized where appropriate for analytics


## 🔧 Advanced Email Format Support

parsedmarc-go features **industry-leading email format compatibility**, automatically handling complex report formats from major email service providers:

### 🎯 **Forensic Reports (RUF) - Universal Compatibility**

#### **Plain Text Format** 
Simple feedback reports embedded directly in email body text

#### **MIME Multipart Email Formats** ⭐
**Automatically parsed with full provider compatibility:**

| **Provider** | **Format** | **Encoding** | **Content-Type** |
|--------------|------------|--------------|------------------|
| **LinkedIn** | `multipart/report` | Plain text | `message/feedback-report` |
| **Domain.de** | `multipart/report` | Plain text | `message/feedback-report; name=report` |
| **Netease** | `multipart/mixed` | **Base64** | `message/feedback-report; name="ATT00001"` |
| **Others** | Auto-detected | Base64/Plain | Various MIME types |

**🚀 Advanced Processing Pipeline:**
1. **Multi-line header parsing** - Handles wrapped Content-Type headers
2. **MIME boundary extraction** - Robust parsing of complex boundaries  
3. **Base64 decoding** - Automatic detection and decoding
4. **Content-type detection** - Intelligent format recognition
5. **Fallback mechanisms** - Plain text parsing if MIME fails

### 📧 **SMTP TLS Reports - Next-Generation Support**

#### **Direct JSON Format**
Standard RFC 8460 JSON reports processed natively

#### **Email-Based Reports** ⭐ 
**Advanced multi-stage processing pipeline:**

| **Provider** | **Format** | **Pipeline** | **Content-Type** |
|--------------|------------|--------------|------------------|
| **Google** | `multipart/report` | Base64 → GZIP → JSON | `application/tlsrpt+gzip` |
| **Others** | Auto-detected | Base64 → Compression → JSON | `application/tlsrpt+*` |

**🔄 Processing Pipeline:**
```
Email Input → MIME Parse → Base64 Decode → GZIP Decompress → JSON Parse → Structured Data
```

## 📋 Supported Standards

parsedmarc-go implements the following email authentication and reporting standards with **industry-leading compatibility**:

- **<a href="https://tools.ietf.org/html/rfc7489">RFC 7489</a>** - Domain-based Message Authentication, Reporting, and Conformance (DMARC)
  - Aggregate reports (RUA) with enhanced parsing
  - Policy configuration and validation
  - **🆕 Enhanced error diagnostics with line numbers**
  
- **<a href="https://tools.ietf.org/html/rfc6591">RFC 6591</a>** - Authentication Failure Reporting Using the Abuse Reporting Format
  - Forensic/failure reports (RUF) with MIME support
  - **🆕 Advanced MIME multipart parsing**
  - **🆕 Base64-encoded attachment support**
  
- **<a href="https://tools.ietf.org/html/rfc8460">RFC 8460</a>** - SMTP TLS Reporting
  - TLS connection and policy reporting with email format support
  - **🆕 Email-based reports with compression support**
  - **🆕 Complete ClickHouse schema for analytics**


## 📚 Documentation

### 📖 Table of Contents

#### 🚀 **Getting Started**
- **[📋 Complete Documentation](https://cabonemailserver.github.io/ParseDMarc-go/docs/index)** - Overview and architecture
- **[⚡ Installation](https://cabonemailserver.github.io/ParseDMarc-go/docs/installation)** - Installation and initial setup  
- **[⚙️ Configuration](https://cabonemailserver.github.io/ParseDMarc-go/docs/configuration)** - Detailed configuration options
- **[💡 Usage](https://cabonemailserver.github.io/ParseDMarc-go/docs/usage)** - Usage guide and examples

#### 🗄️ **Database and Visualization**
- **[🗃️ ClickHouse](https://cabonemailserver.github.io/ParseDMarc-go/docs/clickhouse)** - ClickHouse configuration and optimization
- **[📊 Grafana](https://cabonemailserver.github.io/ParseDMarc-go/docs/grafana)** - Dashboards and visualizations
- **[📈 Monitoring](https://cabonemailserver.github.io/ParseDMarc-go/docs/monitoring)** - Prometheus metrics and monitoring

#### 🔌 **API and Integrations**  
- **[🌐 HTTP API](https://cabonemailserver.github.io/ParseDMarc-go/docs/api)** - HTTP endpoints and integrations
- **[📧 DMARC Mailing Lists](https://cabonemailserver.github.io/ParseDMarc-go/docs/mailing-lists)** - Mailing list configuration

#### 📖 **Technical References**
- **[🔒 DMARC Specification](https://cabonemailserver.github.io/ParseDMarc-go/docs/dmarc)** - DMARC standards details
- **[🤝 Contributing](docs/contributing.md)** - Project contribution guide

### 🔗 Quick Links
- **[Quick Setup](https://cabonemailserver.github.io/ParseDMarc-go/docs/installation#quick-setup)** - Get started in 5 minutes
- **[API Examples](https://cabonemailserver.github.io/ParseDMarc-go/docs/api#examples)** - Ready-to-use HTTP integrations
- **[Grafana Dashboards](https://cabonemailserver.github.io/ParseDMarc-go/docs/grafana#dashboards)** - Pre-configured visualizations
- **[Prometheus Metrics](https://cabonemailserver.github.io/ParseDMarc-go/docs/monitoring#metrics)** - Complete monitoring

> 💡 **Tip**: Start with the [complete documentation](https://cabonemailserver.github.io/ParseDMarc-go/docs/index) for an overview, then check the [installation guide](https://cabonemailserver.github.io/ParseDMarc-go/docs/installation) to get started quickly.

## License

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **[Sean Whalen](https://github.com/seanthegeek)** for the original Python [parsedmarc](https://github.com/domainaware/parsedmarc) project
- **[Claude AI](https://claude.ai)** for comprehensive Go conversion and advanced feature development

---

**📞 Issues**

- [GitHub Issues](https://cabonemailserver.github.io/ParseDMarc-go.git/issues)


