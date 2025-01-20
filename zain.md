# 1: Extracting Doctor Details with single location and also for multiple location 

## Key points

1. **Handler Matchers**: Matches the correct handler to be applied based on the presence of the doctor’s location.
2. **Field Group Before Row Selector In Extract Multiple Rows**: By placing the extraction fields (e.g., doctor name, gender, specialty, etc.) before the row_selector, the same set of doctor-level details are extracted once per page, and then location-level details are extracted for each location the doctor is associated with.

Below template handlers are taken as a example from this config `here <https://github.com/crawlnow/crawl-packages/blob/main/packages/durableca-Nate/Priviahealth/config.yaml>`_

## Template Structure Overview

- **Template Name**: priviahealth  
- **Domain**: priviahealth.com  
- **Feed**: Contains `required_fields` and `export_fields` that define which data points are to be extracted.  
- **Handler Matchers**: Matches the correct handler to be applied based on the presence of the doctor’s location.  
- **Matchers**: Defines how to identify pages with or without location information (using XPath selectors).  
- **Retry Policies**: Defines the retry behavior when certain HTTP errors occur or when a match cannot be found.  
- **Handlers**: Defines actions to extract doctor data in two different scenarios (with and without location).  

---

## Handlers

There are two handlers used in this template:

### **Handler 1: doctor_detail_page_with_location_handler**

**Trigger**: This handler is activated when the XPath selector matches a page with location information.  
**Action**: Extracts data in multiple rows to handle multiple locations for a single doctor.  
**Explanation**:
- **Action Type**: `extract_multiple_rows` – This allows the extraction of multiple locations for the same doctor, ensuring that if a doctor is associated with more than one location, all those locations are captured.
- **Fields Group**: Defines the fields to be extracted, including doctor name, gender, language, medical specialty, practice name, services, education, insurance, and location-specific fields (e.g., location name, street address, city, state, zip code, phone, fax).
- **Row Selector**: The XPath `//li[@class="provider-profile-provider-location css-3tglrm"]` is used to identify each row representing a doctor’s location on the detail page.
- **Field Group Before Row Selector**: By placing the extraction fields (e.g., doctor name, gender, specialty, etc.) before the row_selector, the same set of doctor-level details are extracted once per page, and then location-level details are extracted for each location the doctor is associated with.

### **Handler 2: doctor_detail_page_without_location_handler**

**Trigger**: This handler is activated when the XPath selector for location does not match (i.e., no location information is present).  
**Action**: Extracts a single row of data since there is only one location or no location information.  
**Explanation**:
- **Action Type**: `extract_single_row` – This is used to extract data in a single row, as there is no location data to handle multiple rows.
- **Fields Group**: Same as the previous handler, but only extracted once since there are no multiple locations to process.
- **Row Selector**: Not applicable in this case because there is no location-level detail to handle.

---

## Explanation of Extracting Multiple Locations for a Single Doctor

In the case where a doctor has multiple locations, the `doctor_detail_page_with_location_handler` is used to extract data for each location. This is accomplished using the following steps:

### **Fields Group**:
- The extraction for doctor-level information (e.g., name, specialty, education, etc.) is placed before the row-level selector. This means that the doctor’s general details are extracted once for the page.

### **Row Selector**:
- The XPath `//li[@class="provider-profile-provider-location css-3tglrm"]` identifies each individual location. Each time this XPath matches an element (i.e., a location), the data defined in the `row_level_fields_group` (such as location name, street address, city, state, zip code, phone, fax) will be extracted as part of the result.

### **Row Level Fields Group**:
- For each location row, the following fields are extracted:
  - Location Name
  - Street Address
  - City
  - State
  - Zip Code
  - Location Phone
  - Fax
  
The result is a list of locations for the doctor, with each row containing both the doctor’s general information (extracted once) and the specific location information (extracted for each location).

By placing the doctor’s general information in the `fields_group` before the `row_selector`, we ensure that the doctor-level data is extracted only once, and then the location-specific data is extracted for each location, making it possible to handle multiple locations for a single doctor.

---

## How to Extract Multiple Locations for a Single Doctor

To summarize how to extract multiple locations for a single doctor:

- **Before the Row Selector**:  
   The general doctor information (e.g., doctor name, gender, language, specialty, etc.) is extracted in the `fields_group`. These fields are extracted once per page.

- **Using Row Selector**:  
   The XPath `//li[@class="provider-profile-provider-location css-3tglrm"]` identifies each individual location. Each location will be extracted separately.

- **Location-Specific Fields**:  
   In the `row_level_fields_group`, each location will have its own set of fields (e.g., location name, address, phone number, etc.), and these will be populated for each matched location.

The result is a comprehensive list of locations for each doctor.


By structuring the template in this way, we can ensure efficient extraction of both doctor details and location details, even when multiple locations are associated with a single doctor.


# 2: Using Follow Requests with Extract Multiple Rows

This guide explains how to use follow requests in an **Extract Multiple Rows** action, where the row-level script function generates data dynamically. Follow requests in **extract_multiple_rows** action enable fetching data in parallel using URLs and bodies provided in each row, facilitating efficient pagination and data extraction.

---

## **Key Concepts**

### **1. Extract Multiple Rows**
- Extracts data from a source, such as URLs, request bodies, or metadata, into rows.
- Each row contains fields, such as `URL` and `body`, required for subsequent follow requests.

### **2. Script Function**
- Runs **at the row level** within `Extract Multiple Rows`.
- Dynamically generates data (e.g., pagination URLs and bodies).
- Outputs multiple rows, each containing the necessary data for follow requests.

### **3. Follow Request**
- Executes HTTP requests for each row independently.
- Configurable with:
  - **URL**: Derived from the row (e.g., `{{game_link_follow}}`).
  - **Body**: Dynamic, row-specific data (e.g., `{{game_link_follow_body}}`).
  - **Headers**: Defined globally or per-request as needed.

---

## **Code Example**

```yaml
      - name: item_page_handler
        actions:
          - action_type: extract_multiple_rows
            row_selector:
              language: json_path
              path:
                - 'data'
            row_level_fields_group:
              script_function: get_next_page_body
              follow_requests:
                - url: '{{next_page_url_}}'
                  method: POST
                  body: '{{next_page_body_}}'
                  headers:
                    authority: 'www.stubhub.com'
                    accept: '*/*'
                    accept-language: 'en-GB,en-US;q=0.9,en;q=0.8'
                    content-type: 'application/json'
                    origin: 'https://www.stubhub.com'
                    request-context: 'appId=cid-v1:d2ec73bc-3fa1-4bd1-8c0c-c27c69bb4833'
                    request-id: '|9549467d856042ed921be3e93fe0ce0d.1f2aa1b4a1fa4cb9'
                    sec-ch-ua: '"Chromium";v="130", "Google Chrome";v="130", "Not?A_Brand";v="99"'
                    sec-ch-ua-mobile: '?0'
                    sec-ch-ua-platform: '"Windows"'
                    sec-fetch-dest: 'empty'
                    sec-fetch-mode: 'cors'
                    sec-fetch-site: 'same-origin'
                    traceparent: '00-9f57ad9def5645e0991d408d0a15d223-e0dbeccd65644dd7-01'
              fields:
                - name: next_page_url_
                  selector:
                    language: json_path
                    path:
                      - 'nextPageUrl'
                - name: next_page_body_
                  selector:
                    language: json_path
                    path:
                      - 'nextPageBody'        
          - action_type: extract_multiple_rows
            row_selector:
              language: json_path
              path:
                - 'Items'
                - 'items'
            row_level_fields_group:
              fields:
                - name: price_per_each
                  selector:
                    language: json_path
                    path:
                      - 'RawPrice'
                      - 'rawPrice'
```

---

## **Advantages**

1. **Independent Pagination Requests**:
   - Normally, pagination requires sequential requests where each page depends on the previous one.
   - Here, the script function generates all the required `URL` and `body` data at once, enabling follow requests to run **independently and concurrently**.

2. **Parallel Execution**:
   - Multiple requests for subsequent pages can run simultaneously.
   - Reduces dependency and speeds up the extraction process.


---

## **Example Use Case**

- **Scenario**: Scraping event data from an API where:
  - Each page provides a list of event URLs and request bodies.
  - Script function dynamically generates the next set of requests.
  - Follow requests independently fetch additional event details for each page.

---

By combining `Extract Multiple Rows`, `Script Function`, and `Follow Requests`, you can efficiently handle paginated and dynamic data extraction scenarios in those case where Supports scenarios where next-page requests data can be pre-generated at once for all pages (e.g., APIs or paginated web structures). This approach minimizes dependencies, maximizes parallel processing, and ensures robust data retrieval.


# 3: Triggering Handlers Based on URL Patterns in Follow Requests

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

# 3. Pagination 

We have a action written specifically for pagination.

    ```
              - action_type: paginate_next_page_button
            selector:
              language: xpath
              path:
                - //a[@id="next-button"]/@data-url
    ```

This task automates pagination by interacting with the "Next Page" button until the last page is reached.

### Configuration:
- **Action Type:** `paginate_next_page_button`
- **Selector:**
  - **Language:** `xpath`
  - **Path:**
    ```
    - //a[@id="next-button"]/@data-url
    ```

### Functionality:
The task extracts the provided XPath or JSON path of the "Next Page" button and continues clicking it to navigate through pages until no further pages are available.

### Advanced Configuration:
In cases where direct interaction with the "Next Page" button is not feasible (e.g., URL generation based on dynamic parameters like an ID or start index), you can use a script function to generate the next page URL dynamically.

- **Code Example:**
  ```yaml
  - action_type: paginate_next_page_button
    script_function: get_next_page_url
    selector:
      language: json_path
      path:
        - next_page_url
  ```

- **Script Function Example:**
  ```python
  def get_next_page_url(response):

      json_obj = response.json()
      total_records = json_obj['data']['paging']['total']
      start = json_obj['data']['paging']['start']
      count = json_obj['data']['paging']['count']

      if start + count >= total_records:
          return {'next_page_url': None}, 'json'

      url = response.url.replace(f'start={start}', f'start={start+10}')
      return {'next_page_url': url}, 'json'
  ```

### How It Works:
-  The script function generates the next page URL dynamically by modifying required parameters (e.g., start index).
-  The URL is returned as a JSON dictionary (e.g., `{'next_page_url': url}`).
-  In the configuration file, a selector with `json` language is added, and the `path` specifies the key of the value returned by the script function (e.g., `next_page_url`).
-  Pagination happens seamlessly by using the dynamically generated URLs.

For advanced cases, the script function allows dynamic URL generation by modifying required parameters.


### POST Request Pagination:

For tasks involving POST requests, you can use the follow_requests method in the YAML configuration. This approach allows dynamic pagination by modifying the request payload and URL.

- **Code Example:**
  ```yaml
  - script_function: pagination
  follow_requests:
    - url: '{{url}}'
      method: POST
      body: '{{payload}}'
  ```

- **Script Function Example:**
  ```python
  def pagination(response):
    text: str = response.text
    json_text = json.loads(text)

    payload_json = json.loads(response.request.body.decode('utf-8'))
    try:
        total_records = json_text['data']['search']['resultList']['totalNumRecs']
        cursor = json_text['data']['search']['resultList']['firstRecNum'] + json_text['data']['search']['resultList']['lastRecNum']

        if cursor < total_records:
            payload_json['variables']['no'] = json_text['data']['search']['resultList']['firstRecNum'] + 100
            updated_url = response.url
        else:
            updated_url = None
            payload_json = None

        pagination_dict = {'url': updated_url, 'body': json.dumps(payload_json)}
    except:
        updated_url = None
        payload_json = None
        pagination_dict = {'url': updated_url, 'body': json.dumps(payload_json)}

    return pagination_dict, 'json'
  ```

### How It Works:
-  The script function extracts pagination data from a JSON response, calculates whether there are more records to fetch, and updates the request payload with the new parameters (if applicable). If there are more records, it modifies the no field in the request body to fetch the next set of records and returns the updated URL and payload. Otherwise, it returns None for both, indicating no further pages.
-  The updated URL and payload are returned as a JSON dictionary (e.g., {'url': updated_url, 'body': payload}).
-  In the configuration file, the follow_requests section uses the dynamically generated URL and payload for seamless pagination.
