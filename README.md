# LogBench

A powerful TUI (Terminal User Interface) for analyzing Rails application logs in real-time. LogBench provides an intuitive interface to view HTTP requests, SQL queries, and performance metrics from your Rails logs.

![LogBench Preview](logbench-preview.png)

## Features

- 🚀 **Real-time log analysis** with auto-scroll
- 📊 **Request correlation** - groups SQL queries with HTTP requests
- 🔍 **Advanced filtering** by method, path, status, controller, and more
- 📈 **Performance insights** - duration, allocations, query analysis
- 🎨 **Beautiful TUI** with syntax highlighting and ANSI color support
- ⚡ **Fast parsing** of JSON-formatted logs

## Installation

Add LogBench to your Rails application's Gemfile:

```ruby
# Gemfile
group :development do
  gem 'log_bench'
end
```

Then run:

```bash
bundle install
```

## Configuration

LogBench is **automatically enabled in development environment** for backward compatibility. This gem heavily relies on [lograge](https://github.com/roidrage/lograge) gem for request tracking, we add lograge to your Rails application and configure it automatically, but you can also configure lograge manually if you want.

### Default Behavior

LogBench works out of the box! Just add the gem and restart your Rails server.

It automatically configures:
- Lograge with JSON formatter and custom options
- Rails logger with LogBench::JsonFormatter
- Request ID tracking via LogBench::Current
- Automatic injection of request_id into ApplicationController

No configuration needed in development!

### Custom Configuration (Optional)

To customize LogBench behavior, create `config/initializers/log_bench.rb`:

```ruby
# config/initializers/log_bench.rb
if defined?(LogBench)
  LogBench.setup do |config|
    # Enable/disable LogBench (default: true in development, false elsewhere)
    config.enabled = Rails.env.development? # or any other condition
  
    # Disable automatic lograge configuration (if you want to configure lograge manually)
    # config.configure_lograge_automatically = false  # (default: true)

    # Customize initialization message
    # config.show_init_message = :min # :full, :min, or :none (default: :full)
  
    # Specify which controllers to inject request_id tracking
    # config.base_controller_classes = %w[CustomBaseController] # (default: %w[ApplicationController, ActionController::Base])
  end
end
```

### Manual Lograge Configuration

If you already have lograge configured or want to manage it manually:

```ruby
# config/initializers/log_bench.rb
if defined?(LogBench)
  LogBench.setup do |config|
    # ... other config ...
    config.configure_lograge_automatically = false  # Don't touch my lograge config!
  end
end

# Then configure lograge yourself in config/environments/development.rb or an initializer, 

Rails.application.configure do
  config.lograge.enabled = true
  config.lograge.formatter = Lograge::Formatters::Json.new
  config.lograge.custom_options = lambda do |event|
    # Your custom lograge configuration
    params = event.payload[:params]&.except("controller", "action")
    { params: params } if params.present?
  end
end
```
For more information about lograge configuration, see [Lograge's documentation](https://github.com/roidrage/lograge).

## Usage

### Basic Usage

View your development logs:

```bash
log_bench
# or explicitly for a specific log file
log_bench log/development.log
```

### TUI Controls

- **Navigation**: `↑↓` or `jk` to navigate requests
- **Pane switching**: `←→` or `hl` to switch between request list and details
- **Filtering**: `f` to open filter dialog
- **Clear filter**: `c` to clear an active filter (press `escape` or `enter` before pressing `c` to clear)
- **Sorting**: `s` to cycle through sort options (timestamp, duration, status)
- **Auto-scroll**: `a` to toggle auto-scroll mode
- **Copy**: `y` to copy the selected item to clipboard (request details or SQL query)
- **Text selection**: `t` to toggle text selection mode (enables mouse text selection)
- **Clear requests**: `Ctrl+L` to clear all requests from memory (preserves current position)
- **Undo clear**: `Ctrl+R` to restore previously cleared requests + any new requests (restores exact position)
- **Quit**: `q` to exit

### Filtering

Press `f` to open the filter dialog.

In the left pane you can filter by:

- **Method**: GET, POST, PUT, DELETE, etc.
- **Path**: URL path patterns
- **Status**: HTTP status codes (200, 404, 500, etc.)
- **Controller**: Controller name
- **Action**: Action name
- **Request ID**: Unique request identifier

Examples:
- Filter by method: `GET`
- Filter by path: `/api/users`
- Filter by status: `500`
- Filter by controller: `UsersController`
- Filter by action: `create`
- Filter by request ID: `abcdef-b1n2mk ...`

In the right pane you can filter related log lines by text content to find specific SQL queries or anything else
you want to find in the logs.

### Copying Content

LogBench provides multiple ways to copy content:

**Smart Copy with `y` key:**
- **Left pane**: Copies complete request details (method, path, status, duration, etc.)
- **Right pane**: Copies the selected SQL query with its call source location

**Text Selection Mode:**
- Press `t` to toggle text selection mode
- When enabled, you can use your mouse to select and copy text normally
- When disabled, mouse clicks navigate the interface

**Clipboard Support:**
- **macOS**: Uses `pbcopy` (built-in)
- **Linux**: Uses `xclip` or `xsel` (install with your package manager)
- **Fallback**: Saves to `/tmp/logbench_copy.txt` if no clipboard tool available


## Log Format

LogBench works with JSON-formatted logs. Each log entry should include:

**Required fields for HTTP requests:**
- `method`: HTTP method (GET, POST, etc.)
- `path`: Request path
- `status`: HTTP status code
- `request_id`: Unique request identifier
- `duration`: Request duration in milliseconds

**Optional fields:**
- `controller`: Controller name
- `action`: Action name
- `allocations`: Memory allocations
- `view`: View rendering time
- `db`: Database query time

**Other query logs:**
- `message`: SQL query with timing information
- `request_id`: Links query to HTTP request

## Testing

LogBench includes a comprehensive test suite to ensure reliability and correctness.

### Running Tests

```bash
# Run all tests
bundle exec rake test

# Run specific test files
bundle exec ruby test/test_log_entry.rb
bundle exec ruby test/test_request.rb
bundle exec ruby test/test_json_formatter.rb

# Run tests with verbose output
bundle exec rake test TESTOPTS="-v"
```

### Test Coverage

The test suite covers:

- **Log parsing**: JSON format detection and parsing
- **Request correlation**: Grouping SQL queries with HTTP requests
- **Filtering**: Method, path, status, and duration filters
- **JsonFormatter**: Custom logging format handling
- **TUI components**: Screen rendering and user interactions
- **Edge cases**: Malformed logs, missing fields, performance scenarios

### Writing Tests

When contributing, please include tests for new features:

```ruby
# test/test_new_feature.rb
require "test_helper"

class TestNewFeature < Minitest::Test
  def test_feature_works
    # Your test code here
    assert_equal expected, actual
  end
end
```

## Troubleshooting

### No requests found

1. **Check log file exists**: Ensure the log file path is correct
2. **Verify lograge configuration**: Make sure lograge is enabled and configured
3. **Check log format**: LogBench requires JSON-formatted logs
4. **Generate some requests**: Make HTTP requests to your Rails app to generate logs

### SQL queries not showing

1. **Check request_id correlation**: Ensure SQL queries and HTTP requests share the same `request_id`
2. **Verify Current model**: Make sure `Current.request_id` is being set properly
3. **Check JsonFormatter**: Ensure the JsonFormatter is configured for your Rails logger

### Performance issues

1. **Large log files**: LogBench loads the entire log file into memory. For very large files, consider rotating logs more frequently
2. **Real-time parsing**: Use auto-scroll mode (`a`) for better performance with actively growing log files

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This gem is available as open source under the terms of the [MIT License](LICENSE).

## Support

- 🐛 **Bug reports**: [GitHub Issues](https://github.com/silva96/log_bench/issues)
