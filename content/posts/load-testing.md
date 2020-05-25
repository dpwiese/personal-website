---
title: "Load Testing With Load Impact"
date: 2020-05-25T14:00:00-04:00
draft: true
toc: false
# ADD OG IMAGE WITH THIS POST:
images: ["img/posts/load-testing/og-image.png"]
tags: 
  - loadimpact
  - programming
  - lua
keywords: [loadimpact, programming, lua]
---

# Introduction

Circle CI `config.yml`

```yaml
version: 2
jobs:
  build:
    shell: /bin/bash --login
    docker:
      - image: circleci/build-image:ubuntu-14.04-XXL-upstart-1189-5614f37
        command: /sbin/init
    steps:
      - checkout
      - run:
          name: install dependencies
          command: |
            sudo apt-get install lua5.2
            sudo apt-get install luarocks
            sudo luarocks install luacheck 0.22.0-1
      - run:
          name: run tests
          command: luacheck *.lua
```

https://loadimpact.com/load-script-api#http-request

http://support.loadimpact.com/knowledgebase/articles/814794-testing-a-system-that-requires-uploading-a-file

```lua
-- Set test target endpoint
local test_target = "staging"

-- Config for different API endpoints
local config = {
  production = {
    url = "https://api.danielwiese.com/graph",
    token = "<production-token>"
  },
  staging = {
    url = "https://stagingapi.danielwiese.com/graph",
    token = "<staging-token>"
  },
}

-- Set HTTP request headers
local headers = {
    ["Content-Type"] = "application/json",
    ["Authorization"] = "Token token=" .. config[test_target].token
}

-- Create data to send with request
local data = [[{ "query" : "query { friends (limit: 100) { id, email_address, phone_number } }" }]]

-- Sent HTTP request to API
local response = http.request({"POST",
        config[test_target].url,
        headers = headers,
        data = data,
        response_body_bytes=1024000})

if response.status_code ~= 200 then
  log.error('API error')
  do return end
else
  log.info(response.body)
end
```
