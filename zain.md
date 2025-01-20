## Pagination 

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
1. The script function generates the next page URL dynamically by modifying required parameters (e.g., start index).
2. The URL is returned as a JSON dictionary (e.g., `{'next_page_url': url}`).
3. In the configuration file, a selector with `json` language is added, and the `path` specifies the key of the value returned by the script function (e.g., `next_page_url`).
4. Pagination happens seamlessly by using the dynamically generated URLs.

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
1. The script function extracts pagination data from a JSON response, calculates whether there are more records to fetch, and updates the request payload with the new parameters (if applicable). If there are more records, it modifies the no field in the request body to fetch the next set of records and returns the updated URL and payload. Otherwise, it returns None for both, indicating no further pages.
2. The updated URL and payload are returned as a JSON dictionary (e.g., {'url': updated_url, 'body': payload}).
3. In the configuration file, the follow_requests section uses the dynamically generated URL and payload for seamless pagination.
