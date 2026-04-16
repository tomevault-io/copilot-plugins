## scratchor-web-crawler

> This is a specialized web crawler for extracting educational content from Scratchor platform (https://tiku.scratchor.com/). The project uses Scrapy framework and follows ethical crawling practices.

# Scratchor Web Crawler - Cursor Rules

## Project Overview
This is a specialized web crawler for extracting educational content from Scratchor platform (https://tiku.scratchor.com/). The project uses Scrapy framework and follows ethical crawling practices.

## Development Guidelines

### Code Style
- Use English for all file names, variable names, and comments
- Follow PEP 8 Python style guide
- Use meaningful variable and function names
- Add docstrings for all classes and functions
- Keep functions small and focused (max 50 lines)

### File Organization
- All files and directories must use English names
- Follow the established project structure
- Keep related functionality grouped together
- Use clear, descriptive file names

### Scrapy Best Practices
- Always respect robots.txt
- Use appropriate delays between requests (1-3 seconds)
- Implement proper error handling and retry mechanisms
- Log important events and errors
- Use Item classes for structured data
- Implement data validation in pipelines

### Anti-Detection Measures
- Rotate User-Agent strings
- Use random delays between requests
- Implement respectful crawling patterns
- Monitor response codes and adjust behavior
- Never overwhelm the target server

### Data Handling
- Validate all extracted data
- Handle missing or malformed data gracefully
- Use appropriate data types in Item definitions
- Implement proper data cleaning and normalization
- Store data in structured formats (JSON, CSV)

### Error Handling
- Catch and log all exceptions
- Implement retry logic for transient failures
- Provide meaningful error messages
- Use appropriate logging levels
- Handle network timeouts gracefully

### Testing
- Test spiders with small datasets first
- Validate data extraction logic
- Test error handling scenarios
- Monitor memory usage and performance
- Verify output data quality

### Documentation
- Keep README.md updated with current functionality
- Document all configuration options
- Provide clear usage examples
- Document any special requirements or limitations
- Update documentation when adding new features

### Git Workflow
- Use meaningful commit messages
- Create feature branches for new functionality
- Keep commits focused and atomic
- Update version numbers appropriately
- Tag releases properly

### Performance Considerations
- Monitor memory usage during crawling
- Optimize selectors for better performance
- Use appropriate concurrency settings
- Implement data streaming for large datasets
- Profile code for bottlenecks

### Security
- Never commit sensitive credentials
- Use environment variables for configuration
- Validate all user inputs
- Sanitize data before storage
- Follow secure coding practices

### Maintenance
- Regularly update dependencies
- Monitor for website structure changes
- Clean up temporary files and logs
- Archive old data appropriately
- Keep the codebase clean and organized

## Specific to This Project

### Target Website
- Primary target: https://tiku.scratchor.com/
- Respect their terms of service
- Monitor for anti-bot measures
- Adapt to website changes

### Data Types
- Questions: Educational questions with multiple choice answers
- Categories: Subject categories and subcategories
- Images: Question-related images and diagrams
- Metadata: Creation dates, difficulty levels, tags

### Output Requirements
- JSON format for structured data
- Organized directory structure for images
- Comprehensive logging for debugging
- Progress tracking and statistics

### Chinese Network Environment
- Consider network restrictions and latency
- Use appropriate timeout settings
- Implement robust retry mechanisms
- Test with actual network conditions

## Commands Reference

### Development
```bash
# Install dependencies
pip install -r requirements.txt

# Run spider
scrapy crawl scratchor_spider

# Debug mode
scrapy crawl scratchor_spider -s LOG_LEVEL=DEBUG

# Custom settings
scrapy crawl scratchor_spider -s DOWNLOAD_DELAY=2
```

### Testing
```bash
# Test with limited pages
scrapy crawl scratchor_spider -s CLOSESPIDER_PAGECOUNT=10

# Test specific functionality
python -m pytest tests/

# Check code style
flake8 scratchor_crawler/
```

### Maintenance
```bash
# Clean cache
rm -rf .scrapy/

# Update dependencies
pip freeze > requirements.txt

# Check logs
tail -f logs/scrapy.log
```

Remember: Always prioritize ethical crawling practices and respect the target website's resources and policies. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gary0828) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
