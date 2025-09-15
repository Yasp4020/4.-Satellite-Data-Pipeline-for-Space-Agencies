Satellite Data Pipeline REST API Documentation
Version: 3.2.1
Base URL: https://api.satellite-data-pipeline.space/v3
Protocol: HTTPS
Authentication: JWT Bearer Token / API Key

API Overview
The Satellite Data Pipeline REST API provides comprehensive programmatic access to satellite data ingestion, processing, and retrieval services for space agencies and research institutions. This enterprise-grade API supports petabyte-scale data operations with mission-critical reliability and security.

API Standards Compliance
OpenAPI Specification: 3.1.0

HTTP Methods: GET, POST, PUT, DELETE, PATCH

Content Types: application/json, application/xml, multipart/form-data

Character Encoding: UTF-8

Rate Limiting: Sliding window with burst capacity

Versioning: URI versioning with backward compatibility

Base Configuration

openapi: 3.1.0
info:
  title: Satellite Data Pipeline API
  description: Enterprise satellite data management and processing services
  version: 3.2.1
  contact:
    name: API Support Team
    email: api-support@satellite-data-pipeline.space
    url: https://docs.satellite-data-pipeline.space
  license:
    name: Apache 2.0 with Space Agency Extensions
    url: https://www.apache.org/licenses/LICENSE-2.0
servers:
  - url: https://api.satellite-data-pipeline.space/v3
    description: Production API Server
  - url: https://staging-api.satellite-data-pipeline.space/v3
    description: Staging Environment
security:
  - BearerAuth: []
  - ApiKeyAuth: []
Authentication
Security Schemes

components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT token obtained from authentication endpoint
    ApiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
      description: API key for programmatic access
    OAuthFlow:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://auth.satellite-data-pipeline.space/oauth/authorize
          tokenUrl: https://auth.satellite-data-pipeline.space/oauth/token
          scopes:
            read:data: Read access to satellite data
            write:data: Write access for data ingestion
            admin:system: Administrative access to system resources
Authentication Endpoints
POST /auth/login
Authenticate user credentials and obtain JWT token.

Request Body:

json
{
  "username": "scientist@space-agency.gov",
  "password": "secure_password",
  "mfa_token": "123456",
  "client_info": {
    "ip_address": "192.168.1.100",
    "user_agent": "SatelliteDataClient/1.0",
    "device_fingerprint": "abc123def456"
  }
}
Response 200:

json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "read:data write:data",
  "user_info": {
    "user_id": "usr_12345",
    "username": "scientist@space-agency.gov",
    "organization": "NASA",
    "security_clearance": "SECRET",
    "roles": ["scientist", "data_analyst"]
  }
}
Error Responses:

json
// 401 Unauthorized
{
  "error": "invalid_credentials",
  "error_description": "Invalid username or password",
  "error_code": "AUTH_001",
  "timestamp": "2025-09-15T12:30:00Z"
}

// 403 MFA Required
{
  "error": "mfa_required",
  "error_description": "Multi-factor authentication required",
  "error_code": "AUTH_002",
  "mfa_challenge": {
    "challenge_id": "mfa_789",
    "methods": ["totp", "sms"],
    "expires_in": 300
  }
}
POST /auth/refresh
Refresh expired JWT tokens using refresh token.

Request Body:

json
{
  "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
}
POST /auth/logout
Invalidate current session and tokens.

Request Headers:

text
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
Data Ingestion Endpoints
POST /ingestion/satellite-feeds
Submit satellite telemetry data for processing.

Authentication: Required (write:data scope)
Rate Limit: 1000 requests/minute per satellite

Request Headers:

text
Content-Type: multipart/form-data
Authorization: Bearer <jwt_token>
X-Satellite-ID: SAT-12345
X-Data-Type: telemetry
X-Priority: high
Request Body (Multipart):

json
{
  "metadata": {
    "satellite_id": "SAT-12345",
    "ground_station": "Svalbard-01",
    "acquisition_time": "2025-09-15T12:30:00Z",
    "frequency_band": "X-band",
    "data_rate": "2.5 Gbps",
    "orbit_number": 12847,
    "data_classification": "UNCLASSIFIED",
    "quality_score": 0.98
  },
  "payload": "<binary_data_file>",
  "checksum": {
    "algorithm": "SHA-256",
    "value": "a1b2c3d4e5f6..."
  }
}
Response 202:

json
{
  "ingestion_id": "ing_789abc123",
  "status": "accepted",
  "estimated_processing_time": 120,
  "tracking_url": "/ingestion/status/ing_789abc123",
  "data_preview": {
    "size_bytes": 2147483648,
    "records_estimated": 1500000,
    "time_range": {
      "start": "2025-09-15T12:00:00Z",
      "end": "2025-09-15T12:30:00Z"
    }
  }
}
GET /ingestion/status/{ingestion_id}
Check status of data ingestion job.

Path Parameters:

ingestion_id (required): Unique ingestion job identifier

Response 200:

json
{
  "ingestion_id": "ing_789abc123",
  "status": "processing",
  "progress": {
    "percentage": 75,
    "stage": "quality_validation",
    "records_processed": 1125000,
    "records_total": 1500000
  },
  "timing": {
    "submitted_at": "2025-09-15T12:30:00Z",
    "started_at": "2025-09-15T12:30:15Z",
    "estimated_completion": "2025-09-15T12:32:00Z"
  },
  "quality_metrics": {
    "data_integrity_score": 0.98,
    "signal_quality": 0.95,
    "completeness": 0.99
  }
}
POST /ingestion/batch
Submit multiple satellite data files for batch processing.

Request Body:

json
{
  "batch_id": "batch_456def789",
  "files": [
    {
      "file_path": "s3://satellite-data/raw/SAT-12345/2025-09-15/data-001.raw",
      "metadata": {
        "satellite_id": "SAT-12345",
        "acquisition_time": "2025-09-15T12:00:00Z"
      }
    },
    {
      "file_path": "s3://satellite-data/raw/SAT-12345/2025-09-15/data-002.raw",
      "metadata": {
        "satellite_id": "SAT-12345",
        "acquisition_time": "2025-09-15T12:15:00Z"
      }
    }
  ],
  "processing_options": {
    "priority": "normal",
    "validation_level": "strict",
    "output_format": "parquet"
  }
}
Data Retrieval and Querying
GET /data/query
Query satellite data with advanced filtering and pagination.

Authentication: Required (read:data scope)
Rate Limit: 10,000 requests/hour per user

Query Parameters:

text
satellite_id: SAT-12345,SAT-67890
start_date: 2025-09-01T00:00:00Z
end_date: 2025-09-15T23:59:59Z
data_type: imagery,telemetry
bbox: -122.5,37.7,-122.3,37.8
cloud_coverage_max: 20
limit: 100
offset: 0
sort: acquisition_time:desc
format: json
include_preview: true
Response 200:

json
{
  "query_id": "qry_123abc456",
  "total_records": 15742,
  "returned_records": 100,
  "pagination": {
    "limit": 100,
    "offset": 0,
    "next_page": "/data/query?offset=100&limit=100&query_id=qry_123abc456",
    "total_pages": 158
  },
  "data": [
    {
      "record_id": "rec_789def123",
      "satellite_id": "SAT-12345",
      "acquisition_time": "2025-09-15T12:30:00Z",
      "data_type": "multispectral_imagery",
      "location": {
        "center": {
          "latitude": 37.7749,
          "longitude": -122.4194
        },
        "bounds": {
          "north": 37.8049,
          "south": 37.7449,
          "east": -122.3894,
          "west": -122.4494
        }
      },
      "metadata": {
        "cloud_coverage": 15,
        "sun_elevation": 45.2,
        "off_nadir_angle": 12.5,
        "spatial_resolution": 0.5,
        "spectral_bands": 8
      },
      "access_urls": {
        "download": "https://api.satellite-data-pipeline.space/v3/data/download/rec_789def123",
        "preview": "https://cdn.satellite-data-pipeline.space/previews/rec_789def123.jpg",
        "metadata": "https://api.satellite-data-pipeline.space/v3/data/metadata/rec_789def123"
      },
      "data_products": [
        {
          "product_id": "prd_l1a_456",
          "processing_level": "L1A",
          "format": "GeoTIFF",
          "size_bytes": 2147483648,
          "checksum": "sha256:a1b2c3d4..."
        }
      ]
    }
  ]
}
GET /data/download/{record_id}
Download satellite data file with streaming support.

Path Parameters:

record_id (required): Unique data record identifier

Query Parameters:

text
format: original,geotiff,netcdf
compression: none,gzip,lz4
chunk_size: 8388608
range: bytes=0-1048576
Response 200:

text
Content-Type: application/octet-stream
Content-Length: 2147483648
Content-Disposition: attachment; filename="SAT-12345_20250915_123000.tif"
X-Data-Processing-Level: L1A
X-Satellite-ID: SAT-12345
X-Acquisition-Time: 2025-09-15T12:30:00Z
Accept-Ranges: bytes

<binary_data_stream>
POST /data/search
Advanced search with complex filtering criteria.

Request Body:

json
{
  "query": {
    "spatial": {
      "geometry": {
        "type": "Polygon",
        "coordinates": [[
          [-122.5, 37.7],
          [-122.3, 37.7],
          [-122.3, 37.8],
          [-122.5, 37.8],
          [-122.5, 37.7]
        ]]
      },
      "relation": "intersects"
    },
    "temporal": {
      "start": "2025-09-01T00:00:00Z",
      "end": "2025-09-15T23:59:59Z"
    },
    "filters": [
      {
        "field": "cloud_coverage",
        "operator": "<=",
        "value": 20
      },
      {
        "field": "data_type",
        "operator": "in",
        "value": ["imagery", "hyperspectral"]
      }
    ]
  },
  "options": {
    "limit": 50,
    "sort": [
      {"field": "acquisition_time", "order": "desc"},
      {"field": "cloud_coverage", "order": "asc"}
    ],
    "include_statistics": true
  }
}
System Monitoring and Health
GET /health
Basic health check endpoint for load balancer monitoring.

Authentication: None required
Rate Limit: No limit

Response 200:

json
{
  "status": "healthy",
  "timestamp": "2025-09-15T12:30:00Z",
  "version": "3.2.1",
  "environment": "production"
}
GET /health/detailed
Comprehensive system health status.

Authentication: Required (admin scope)

Response 200:

json
{
  "overall_status": "healthy",
  "timestamp": "2025-09-15T12:30:00Z",
  "components": {
    "ingestion_layer": {
      "status": "healthy",
      "metrics": {
        "active_connections": 45,
        "throughput_gbps": 35.2,
        "error_rate": 0.001,
        "queue_depth": 12
      }
    },
    "storage_systems": {
      "status": "healthy",
      "cassandra": {
        "status": "healthy",
        "nodes_active": 24,
        "storage_usage_percent": 67
      },
      "hdfs": {
        "status": "healthy",
        "namenode_status": "active",
        "datanode_count": 150,
        "capacity_tb": 1024
      },
      "object_storage": {
        "status": "healthy",
        "total_objects": 15742859,
        "total_size_pb": 12.5
      }
    },
    "processing_framework": {
      "status": "healthy",
      "spark_clusters": {
        "active_applications": 156,
        "executor_count": 2400,
        "memory_utilization": 0.78
      },
      "flink_clusters": {
        "running_jobs": 23,
        "checkpointing_healthy": true
      }
    },
    "api_gateway": {
      "status": "healthy",
      "request_rate_per_second": 2500,
      "response_time_p95_ms": 45,
      "error_rate": 0.0015
    }
  }
}
GET /metrics
Prometheus-compatible metrics endpoint.

Authentication: Required (monitoring scope)

Response 200:

text
# HELP satellite_data_ingestion_rate Data ingestion rate in MB/s
# TYPE satellite_data_ingestion_rate gauge
satellite_data_ingestion_rate{satellite_id="SAT-12345"} 2048.5

# HELP satellite_data_processing_jobs_total Total number of processing jobs
# TYPE satellite_data_processing_jobs_total counter
satellite_data_processing_jobs_total{status="completed"} 1547892
satellite_data_processing_jobs_total{status="failed"} 1203

# HELP satellite_data_storage_usage_bytes Storage usage in bytes
# TYPE satellite_data_storage_usage_bytes gauge
satellite_data_storage_usage_bytes{tier="hot"} 1.073741824e+14
satellite_data_storage_usage_bytes{tier="warm"} 1.125899907e+15
satellite_data_storage_usage_bytes{tier="cold"} 1.125899907e+16
Data Processing Job Management
POST /processing/jobs
Submit new data processing job.

Authentication: Required (write:processing scope)

Request Body:

json
{
  "job_name": "atmospheric_correction_batch_001",
  "job_type": "atmospheric_correction",
  "input_data": {
    "query": {
      "satellite_id": "SAT-12345",
      "start_date": "2025-09-15T00:00:00Z",
      "end_date": "2025-09-15T23:59:59Z",
      "data_type": "multispectral_imagery"
    }
  },
  "processing_parameters": {
    "algorithm": "MODTRAN",
    "atmospheric_model": "tropical",
    "aerosol_model": "continental",
    "visibility_km": 23.0,
    "water_vapor_cm": 2.5
  },
  "output_configuration": {
    "format": "GeoTIFF",
    "compression": "lzw",
    "output_path": "s3://processed-data/atmospheric-corrected/",
    "metadata_format": "iso19115"
  },
  "execution_options": {
    "priority": "normal",
    "max_execution_time_hours": 24,
    "retry_attempts": 3,
    "resource_requirements": {
      "cpu_cores": 64,
      "memory_gb": 256,
      "gpu_count": 4
    }
  }
}
Response 201:

json
{
  "job_id": "job_abc123def456",
  "status": "queued",
  "estimated_start_time": "2025-09-15T13:00:00Z",
  "estimated_completion_time": "2025-09-15T15:30:00Z",
  "queue_position": 5,
  "tracking_url": "/processing/jobs/job_abc123def456/status"
}
GET /processing/jobs/{job_id}/status
Get detailed status of processing job.

Path Parameters:

job_id (required): Unique job identifier

Response 200:

json
{
  "job_id": "job_abc123def456",
  "job_name": "atmospheric_correction_batch_001",
  "status": "running",
  "progress": {
    "percentage": 65,
    "current_stage": "processing",
    "stages": [
      {
        "name": "validation",
        "status": "completed",
        "duration_seconds": 45
      },
      {
        "name": "processing",
        "status": "running",
        "progress_percentage": 65,
        "estimated_remaining_seconds": 1800
      },
      {
        "name": "output_generation",
        "status": "pending"
      }
    ]
  },
  "execution_details": {
    "started_at": "2025-09-15T13:00:00Z",
    "estimated_completion": "2025-09-15T15:30:00Z",
    "resource_usage": {
      "cpu_cores_allocated": 64,
      "memory_gb_allocated": 256,
      "gpu_count_allocated": 4,
      "current_cpu_usage": 0.87,
      "current_memory_usage": 0.73
    }
  },
  "input_summary": {
    "total_files": 1247,
    "total_size_gb": 2048,
    "files_processed": 810
  },
  "output_summary": {
    "files_generated": 810,
    "total_output_size_gb": 1536,
    "output_location": "s3://processed-data/atmospheric-corrected/job_abc123def456/"
  }
}
DELETE /processing/jobs/{job_id}
Cancel processing job.

Response 200:

json
{
  "job_id": "job_abc123def456",
  "status": "cancelled",
  "cancellation_time": "2025-09-15T13:45:00Z",
  "cleanup_status": "in_progress"
}
GET /processing/jobs
List processing jobs with filtering.

Query Parameters:

text
status: queued,running,completed,failed,cancelled
user_id: usr_12345
start_date: 2025-09-01T00:00:00Z
end_date: 2025-09-15T23:59:59Z
limit: 20
offset: 0
Response 200:

json
{
  "total_jobs": 1547,
  "returned_jobs": 20,
  "jobs": [
    {
      "job_id": "job_abc123def456",
      "job_name": "atmospheric_correction_batch_001",
      "job_type": "atmospheric_correction",
      "status": "completed",
      "submitted_by": "usr_12345",
      "submitted_at": "2025-09-15T13:00:00Z",
      "completed_at": "2025-09-15T15:25:00Z",
      "execution_time_seconds": 8700,
      "resource_usage_summary": {
        "cpu_hours": 556.8,
        "memory_gb_hours": 2227.2,
        "gpu_hours": 34.8
      }
    }
  ]
}
Analytics and Reporting
GET /analytics/usage
Retrieve usage analytics and statistics.

Authentication: Required (read:analytics scope)

Query Parameters:

text
time_range: last_24h,last_7d,last_30d,custom
start_date: 2025-09-01T00:00:00Z
end_date: 2025-09-15T23:59:59Z
granularity: hour,day,week,month
group_by: user,organization,satellite,data_type
Response 200:

json
{
  "time_range": {
    "start": "2025-09-01T00:00:00Z",
    "end": "2025-09-15T23:59:59Z"
  },
  "summary": {
    "total_data_ingested_tb": 1247.8,
    "total_data_downloaded_tb": 567.3,
    "total_processing_jobs": 15647,
    "total_api_requests": 2547891,
    "unique_users": 1247,
    "active_satellites": 47
  },
  "data_ingestion": {
    "by_satellite": [
      {
        "satellite_id": "SAT-12345",
        "data_volume_tb": 247.5,
        "successful_passes": 156,
        "failed_passes": 2,
        "average_data_quality": 0.97
      }
    ],
    "by_time": [
      {
        "timestamp": "2025-09-15T00:00:00Z",
        "data_volume_gb": 2048.5,
        "successful_ingestions": 89,
        "failed_ingestions": 1
      }
    ]
  },
  "user_activity": {
    "by_organization": [
      {
        "organization": "NASA",
        "user_count": 247,
        "api_requests": 1247891,
        "data_downloaded_tb": 234.7
      }
    ],
    "top_users": [
      {
        "user_id": "usr_12345",
        "username": "scientist@nasa.gov",
        "api_requests": 15647,
        "data_downloaded_gb": 1247.8,
        "processing_jobs": 89
      }
    ]
  }
}
POST /analytics/custom-report
Generate custom analytics report.

Request Body:

json
{
  "report_name": "monthly_satellite_performance",
  "time_range": {
    "start": "2025-08-01T00:00:00Z",
    "end": "2025-08-31T23:59:59Z"
  },
  "metrics": [
    "data_ingestion_volume",
    "data_quality_score",
    "processing_job_success_rate",
    "user_engagement"
  ],
  "dimensions": [
    "satellite_id",
    "ground_station",
    "data_type"
  ],
  "filters": {
    "satellite_constellation": ["earth_observation", "weather"],
    "data_classification": ["UNCLASSIFIED", "RESTRICTED"]
  },
  "output_format": "json",
  "delivery_method": "api_response"
}
Response 202:

json
{
  "report_id": "rpt_789ghi012",
  "status": "generating",
  "estimated_completion_time": "2025-09-15T13:05:00Z",
  "download_url": "/analytics/reports/rpt_789ghi012/download"
}
GET /analytics/reports/{report_id}/download
Download generated analytics report.

Response 200:

json
{
  "report_id": "rpt_789ghi012",
  "report_name": "monthly_satellite_performance",
  "generated_at": "2025-09-15T13:05:00Z",
  "time_range": {
    "start": "2025-08-01T00:00:00Z",
    "end": "2025-08-31T23:59:59Z"
  },
  "summary": {
    "total_satellites_analyzed": 47,
    "total_data_volume_tb": 15647.8,
    "average_data_quality": 0.967,
    "processing_success_rate": 0.987
  },
  "detailed_results": {
    "satellite_performance": [
      {
        "satellite_id": "SAT-12345",
        "constellation": "earth_observation",
        "metrics": {
          "data_volume_tb": 1247.8,
          "data_quality_score": 0.98,
          "successful_passes": 894,
          "failed_passes": 12,
          "coverage_percentage": 87.5
        }
      }
    ]
  }
}
Error Handling
Standard Error Response Format
json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": "The 'satellite_id' parameter is required but was not provided",
    "timestamp": "2025-09-15T12:30:00Z",
    "request_id": "req_abc123def456",
    "documentation_url": "https://docs.satellite-data-pipeline.space/errors/VALIDATION_ERROR"
  },
  "validation_errors": [
    {
      "field": "satellite_id",
      "code": "MISSING_REQUIRED_FIELD",
      "message": "This field is required"
    }
  ]
}
HTTP Status Codes
Code	Description	Usage
200	OK	Successful GET request
201	Created	Successful POST request creating resource
202	Accepted	Async operation accepted for processing
204	No Content	Successful DELETE request
400	Bad Request	Invalid request parameters or format
401	Unauthorized	Missing or invalid authentication
403	Forbidden	Insufficient permissions for resource
404	Not Found	Requested resource does not exist
409	Conflict	Resource conflict or constraint violation
422	Unprocessable Entity	Valid request format but business logic error
429	Too Many Requests	Rate limit exceeded
500	Internal Server Error	Unexpected server error
502	Bad Gateway	Upstream service unavailable
503	Service Unavailable	System maintenance or overload
Error Codes Reference
Error Code	Description	Resolution
AUTH_001	Invalid credentials	Verify username and password
AUTH_002	MFA required	Complete multi-factor authentication
AUTH_003	Token expired	Refresh JWT token
DATA_001	Invalid data format	Check data format specifications
DATA_002	Data size limit exceeded	Reduce file size or use batch processing
PROC_001	Processing job failed	Check job logs and retry
RATE_001	Rate limit exceeded	Reduce request frequency or upgrade plan
SYS_001	System maintenance	Retry after maintenance window
Rate Limiting
Rate Limit Headers
All API responses include rate limiting information:

text
X-RateLimit-Limit: 10000
X-RateLimit-Remaining: 9950
X-RateLimit-Reset: 1694781600
X-RateLimit-Window: 3600
Rate Limit Tiers
User Tier	Requests/Hour	Data Transfer/Day	Concurrent Jobs
Basic	1,000	100 GB	5
Professional	10,000	1 TB	25
Enterprise	100,000	10 TB	100
Government	Unlimited	Unlimited	500
This comprehensive API documentation provides enterprise-grade specifications for satellite data pipeline operations, ensuring compliance with space agency requirements and supporting mission-critical data operations.


