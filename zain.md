# README: Triggering Handlers Based on URL Patterns in Follow Requests

## **Overview**
In Some websites you might have a listing page with multiple product links. When a follow request is made to scrape a product page, the appropriate handler is triggered based on the URL pattern of the product page. This ensures the correct processing logic is applied to extract data.

## Template Structure Overview

- **crawl-templates**: Defines the domain and scraping configuration for a website.
- **handler_matchers**: Maps handlers to URL patterns using matchers.
- **matchers**: Defines the patterns used to identify URLs for specific pages.
- **handlers**: Contains the actions to be performed when a specific handler is triggered.

```yaml
crawl-templates:
  - name: postpartner.cz
    domain: postpartner.cz
    javascript: false
    feed:
      export_fields:
        - is_available
        - product_url

    handler_matchers:
      - handler: listing_page_handler
        matcher: listing_page_matcher
      - handler: product_page_handler
        matcher: product_page_matcher

    matchers:
      - name: listing_page_matcher
        parameters:
          is_regex: false
          operator: matches
          pattern:
            - /hledani?query=
        type: url_pattern

      - name: product_page_matcher
        parameters:
          is_regex: false
          operator: matches
          pattern:
            - postpartner.cz/
        type: url_pattern

    handlers:

      - name: listing_page_handler
        actions:
          - action_type: extract_multiple_rows

            row_selector:
              language: xpath
              path:
                - (//div[contains(@class,'commodities')]//article[contains(@class,'commodityBox')])[position()<4]

            row_level_fields_group:
              follow_requests:
                - url: '{{product_url}}'
                  method: GET

              fields:
                - name: product_url
                  selector:
                    language: xpath
                    path:
                      - .//@href

      - name: product_page_handler
        actions:
          - action_type: extract_single_row
            fields_group:
              fields:

                - name: is_available
                  selector:
                    language: xpath
                    path:
                      - //div[contains(@class,'commodityDetail')]//dl[contains(@class,'Availability')]//strong[contains(text(), 'Skladem')]
                  transformations:
                    - type: exists
                      parameters:
                        if_exists: 'Yes'
                        if_not_exists: 'No'
```

### **Explanation of Key Sections**
#### **Matchers**
Matchers define the URL patterns to determine which handler to trigger. For example:
- **Listing Page Matcher**: Matches URLs containing `/hledani?query=`.
- **Product Page Matcher**: Matches URLs containing `postpartner.cz/`.

#### **Handlers**
Handlers specify the actions to perform for a matched URL:
- **Listing Page Handler**: Extracts multiple product links and generates follow requests for each product.
- **Product Page Handler**: Extracts data such as availability status from the product page.

#### **Follow Requests**
The `follow_requests` field in the `listing_page_handler` generates GET requests for product pages using the extracted `product_url` field.

### **Workflow**
- The `listing_page_handler` processes the page, extracts product URLs, and generates follow requests.
- When a follow request is made for a product URL, the `product_page_matcher` triggers the `product_page_handler`.
- The `product_page_handler` processes the product page and extracts the required data.

### **Customization**
- Update the `pattern` fields in matchers to match specific URL structures for your use case.
- Define additional handlers and actions as needed to handle other page types.

### **Troubleshooting**
- **No Handler Triggered**: Ensure the URL matches a defined pattern in the matchers.

This configuration enables precise triggering of handlers based on URL patterns, ensuring efficient and accurate data scraping.
