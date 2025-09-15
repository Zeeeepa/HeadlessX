# POST Endpoints Documentation

This document provides detailed information about all POST endpoints available in HeadlessX v1.2.0 (Modular Architecture).

## Authentication

All API endpoints require authentication via one of these methods:
- Query parameter: `?token=YOUR_SECURE_AUTH_TOKEN`
- Header: `X-Token: YOUR_SECURE_AUTH_TOKEN`
- Header: `Authorization: Bearer YOUR_SECURE_AUTH_TOKEN`

> **Note**: Environment variable changed from `TOKEN` to `AUTH_TOKEN` in v1.2.0

## Modular Architecture Benefits

HeadlessX v1.2.0 features a completely refactored modular architecture:
- **Enhanced Performance**: Optimized browser management and resource usage
- **Better Error Handling**: Structured error responses with correlation IDs
- **Improved Logging**: Request tracing and structured logging
- **Rate Limiting**: Built-in protection against abuse

## Core POST Endpoints

### 1. Full Page Rendering (JSON Response)
```http
POST /api/render?token=YOUR_TOKEN
Content-Type: application/json
```

**Description:** Advanced page rendering with comprehensive options and JSON response

**Request Body:**
```json
{
  "url": "https://example.com",
  "waitUntil": "networkidle",
  "timeout": 60000,
  "extraWaitTime": 10000,
  "userAgent": "custom-user-agent",
  "cookies": [
    {
      "name": "session",
      "value": "abc123",
      "domain": ".example.com"
    }
  ],
  "headers": {
    "X-Custom-Header": "value"
  },
  "viewport": {
    "width": 1920,
    "height": 1080
  },
  "scrollToBottom": true,
  "waitForSelectors": [".content", ".main"],
  "clickSelectors": [".load-more", ".accept-cookies"],
  "removeElements": [".ads", ".popup"],
  "customScript": "document.querySelector('.button').click();",
  "waitForNetworkIdle": true,
  "captureConsole": true,
  "returnPartialOnTimeout": true,
  "fullPage": false,
  "screenshotPath": null,
  "screenshotFormat": "png",
  "pdfPath": null,
  "pdfFormat": "A4"
}
```

**Required Fields:**
- `url`: Target URL to render

**Optional Fields:**
- `waitUntil`: Wait condition ('load', 'domcontentloaded', 'networkidle', 'networkidle0') - default: 'networkidle'
- `timeout`: Main timeout in milliseconds - default: 60000
- `extraWaitTime`: Additional wait time for dynamic content - default: 10000
- `userAgent`: Custom user agent (uses realistic rotation if not provided)
- `cookies`: Array of cookie objects to set
- `headers`: Custom HTTP headers object
- `viewport`: Browser viewport dimensions
- `scrollToBottom`: Scroll to trigger lazy loading - default: true
- `waitForSelectors`: Array of CSS selectors to wait for
- `clickSelectors`: Array of CSS selectors to click
- `removeElements`: Array of CSS selectors to remove
- `customScript`: JavaScript code to execute on the page
- `waitForNetworkIdle`: Wait for network requests to finish - default: true
- `captureConsole`: Capture console logs - default: false
- `returnPartialOnTimeout`: Return partial content on timeout - default: true
- `fullPage`: Take full page screenshot if screenshotPath provided
- `screenshotPath`: Path to save screenshot (server-side)
- `screenshotFormat`: Screenshot format ('png'/'jpeg')
- `pdfPath`: Path to save PDF (server-side)
- `pdfFormat`: PDF paper format

**Response:**
```json
{
  "html": "<html>...</html>",
  "title": "Page Title",
  "url": "https://final-url.com",
  "originalUrl": "https://example.com",
  "consoleLogs": [
    {
      "type": "log",
      "text": "Console message",
      "location": {
        "url": "https://example.com",
        "lineNumber": 123
      }
    }
  ],
  "timestamp": "2025-09-12T12:00:00.000Z",
  "wasTimeout": false,
  "contentLength": 45678,
  "screenshotBuffer": null,
  "pdfBuffer": null,
  "isEmergencyContent": false
}
```

**Example:**
```bash
curl -X POST "https://your-subdomain.yourdomain.com/api/render?token=YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "timeout": 30000,
    "scrollToBottom": true,
    "captureConsole": true,
    "waitForSelectors": [".content"],
    "removeElements": [".ads"]
  }'
```

### 2. Raw HTML Extraction (POST)
```http
POST /api/html?token=YOUR_TOKEN
Content-Type: application/json
```

**Description:** Extract raw HTML with all rendering options, returns HTML directly

**Request Body:** Same as `/api/render` endpoint

**Response Headers:**
- `Content-Type`: text/html; charset=utf-8
- `X-Rendered-URL`: Final URL after redirects
- `X-Page-Title`: Page title
- `X-Timestamp`: Rendering timestamp
- `X-Was-Timeout`: Whether timeout occurred
- `X-Content-Length`: Content length
- `X-Is-Emergency`: Whether emergency extraction was used

**Response:** Raw HTML content

**Example:**
```bash
curl -X POST "https://your-subdomain.yourdomain.com/api/html?token=YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "timeout": 30000,
    "waitForSelectors": [".main-content"]
  }'
```

### 3. Clean Text Extraction (POST)
```http
POST /api/content?token=YOUR_TOKEN
Content-Type: application/json
```

**Description:** Extract clean, readable text content with all rendering options

**Request Body:** Same as `/api/render` endpoint

**Response Headers:**
- `Content-Type`: text/plain; charset=utf-8
- `X-Rendered-URL`: Final URL after redirects
- `X-Page-Title`: Page title
- `X-Content-Length`: Text content length
- `X-Timestamp`: Rendering timestamp
- `X-Was-Timeout`: Whether timeout occurred
- `X-Is-Emergency`: Whether emergency extraction was used

**Response:** Clean text content with intelligent formatting

**Example:**
```bash
curl -X POST "https://your-subdomain.yourdomain.com/api/content?token=YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "removeElements": [".sidebar", ".nav", ".ads"]
  }'
```

### 4. Batch Processing
```http
POST /api/batch?token=YOUR_TOKEN
Content-Type: application/json
```

**Description:** Process multiple URLs with controlled concurrency

**Request Body:**
```json
{
  "urls": [
    "https://example1.com",
    "https://example2.com",
    "https://example3.com"
  ],
  "concurrency": 3,
  "timeout": 60000,
  "waitUntil": "networkidle",
  "scrollToBottom": true,
  "extraWaitTime": 5000,
  "returnPartialOnTimeout": true,
  "outputFormat": "html"
}
```

**Required Fields:**
- `urls`: Array of URLs to process

**Optional Fields:**
- `concurrency`: Number of concurrent requests - default: 3, max: 5
- `timeout`: Timeout per URL - default: 60000
- `waitUntil`: Wait condition for all URLs
- `scrollToBottom`: Scroll for all URLs - default: true
- `extraWaitTime`: Extra wait time for all URLs
- `returnPartialOnTimeout`: Return partial content on timeout
- `outputFormat`: Response format ('html', 'text', 'json') - default: 'html'

**Response:**
```json
{
  "results": [
    {
      "url": "https://example1.com",
      "success": true,
      "html": "<html>...</html>",
      "title": "Page 1",
      "contentLength": 12345,
      "timestamp": "2025-09-12T12:00:00.000Z",
      "renderTime": 2345
    },
    {
      "url": "https://example2.com",
      "success": false,
      "error": "Timeout error",
      "timestamp": "2025-09-12T12:00:05.000Z",
      "renderTime": 60000
    }
  ],
  "summary": {
    "total": 3,
    "successful": 2,
    "failed": 1,
    "totalTime": 125000,
    "averageTime": 41667
  },
  "timestamp": "2025-09-12T12:00:00.000Z"
}
```

**Example:**
```bash
curl -X POST "https://your-subdomain.yourdomain.com/api/batch?token=YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "urls": [
      "https://example1.com",
      "https://example2.com"
    ],
    "concurrency": 2,
    "timeout": 30000,
    "outputFormat": "text"
  }'
```

## Advanced Features

### Human-like Behavior Simulation
All POST endpoints automatically include:
- Realistic user agent rotation (Windows browsers)
- Natural mouse movements and interactions
- Human-like scrolling patterns with easing
- Random timing variations and pauses
- Browser-specific headers and properties

### Anti-Detection Techniques
- 40+ stealth techniques automatically applied
- Webdriver property removal
- Realistic browser fingerprinting
- Natural plugin and MIME type spoofing
- Comprehensive automation indicator removal

### Timeout Handling
- **Primary timeout**: Main operation timeout
- **Partial content recovery**: Returns content even on timeout
- **Emergency extraction**: Fallback method for difficult sites
- **Graceful degradation**: Multiple fallback strategies

### Error Recovery
- Automatic retry with simpler settings on failure
- Emergency content extraction for timeout scenarios
- Partial content return when possible
- Detailed error reporting with context

## Cookie Management

**Cookie Format:**
```json
{
  "name": "cookie_name",
  "value": "cookie_value",
  "domain": ".example.com",
  "path": "/",
  "expires": 1640995200,
  "httpOnly": true,
  "secure": true,
  "sameSite": "Lax"
}
```

**Required Fields:** `name`, `value`
**Optional Fields:** `domain`, `path`, `expires`, `httpOnly`, `secure`, `sameSite`

## Custom Scripts

Execute JavaScript on the page after loading:

```json
{
  "customScript": "window.scrollTo(0, document.body.scrollHeight); document.querySelector('.load-more').click();"
}
```

**Best Practices:**
- Keep scripts simple and focused
- Avoid long-running operations
- Handle errors gracefully
- Don't interfere with page functionality

## Error Handling

**Error Response Format:**
```json
{
  "error": "Error type",
  "details": "Detailed error message",
  "timestamp": "2025-09-12T12:00:00.000Z",
  "url": "https://failed-url.com"
}
```

**Common Errors:**
- `400`: Missing/invalid URL or parameters
- `401`: Invalid/missing authentication token
- `500`: Server error or page loading failure
- `503`: Service temporarily unavailable

## Performance Optimization

### Request Optimization
- Use appropriate timeouts for target sites
- Enable `returnPartialOnTimeout` for better reliability
- Set reasonable `extraWaitTime` values
- Use `waitForSelectors` for specific content

### Batch Processing Tips
- Keep concurrency between 2-5 for stability
- Group similar sites together
- Use shorter timeouts for batch requests
- Monitor server resources

### Resource Management
- HeadlessX automatically manages browser instances
- Requests are queued and processed efficiently
- Memory is cleaned up after each request
- Failed requests don't affect other operations

## Security Considerations

1. **Token Security**: Never expose tokens in client-side code
2. **Input Validation**: All URLs and parameters are validated
3. **Resource Limits**: Automatic limits prevent resource exhaustion
4. **Error Handling**: Sensitive information is not exposed in errors

## Best Practices

1. **Start Simple**: Test with basic options before adding complexity
2. **Use Timeouts Wisely**: Balance completeness vs speed
3. **Handle Errors**: Always check response status and handle failures
4. **Monitor Performance**: Track response times and success rates
5. **Respect Targets**: Follow robots.txt and rate limiting guidelines

## Support

For issues with POST endpoints:
1. Verify authentication token
2. Check request body format (valid JSON)
3. Test with simpler parameters first
4. Check server logs for detailed errors
5. Use `/api/health` to verify server status
6. Create GitHub issues for bugs or feature requests
