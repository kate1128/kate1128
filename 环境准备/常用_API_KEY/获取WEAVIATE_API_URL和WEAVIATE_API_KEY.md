```python
WEAVIATE_API_KEY="your_weaviate_api_key"
WEAVIATE_API_URL="your_weaviate_api_url"
COHERE_API_KEY="your_cohere_api_key"
```

> search("weaviate_api_key怎么申请")
>

Weaviate 是一个开源的向量数据库，支持通过 API 密钥进行身份验证，以确保对数据库的安全访问。

**申请 Weaviate API 密钥的方法如下：**

1. **使用 Weaviate Cloud（WCS）**：
    - **创建实例**：在 Weaviate Cloud 上创建一个新的实例。
    - **获取 API 密钥**：在实例的详情页面，点击“API 密钥”按钮，生成并复制新的 API 密钥。 

> [Manage authentication - Weaviate](https://weaviate.io/developers/wcs/platform/manage-api-keys?utm_source=chatgpt.com) · _weaviate.io_  
To delete an API key, follow these steps: Click the Details button. Click the API keys button. Click the trash-can icon to delete the API key. Retrieve your API keys To retrieve your API keys, follow these steps: Open the WCD console and find the details panel for your cluster. Click the API keys button. Copy the key and store it in a safe ...
>

2. **本地部署 Weaviate**：
    - **启用 API 密钥认证**：在 Weaviate 的配置文件（如 `docker-compose.yml`）中，设置以下环境变量：

```yaml
services:
  weaviate:
    environment:
      AUTHENTICATION_APIKEY_ENABLED: 'true'
      AUTHENTICATION_APIKEY_ALLOWED_KEYS: 'your-secret-key'
      AUTHENTICATION_APIKEY_USERS: 'your-username'
```

这将启用 API 密钥认证，并设置允许的密钥和对应的用户。 

> [Authentication - Weaviate](https://weaviate.io/developers/weaviate/configuration/authentication?utm_source=chatgpt.com) · _Weaviate_  
Weaviate controls access through user authentication via API keys or OpenID Connect (OIDC), with an option for anonymous access. Users can then be assigned different authorization levels, as shown in the diagram below.. For example, a user logging in with the API key jane-secret may be granted administrator permissions, while another user logging in with the API key ian-secret may be granted ...
>

3. **管理 API 密钥**：
    - **创建和删除密钥**：在 Weaviate Cloud 的控制台中，您可以创建新的 API 密钥或删除已有的密钥。
    - **权限设置**：API 密钥可分为只读（ReadOnly）和管理员（Admin）两种类型。根据需要，分配适当的权限以控制对数据库的访问。 

> [Manage authentication - Weaviate](https://weaviate.io/developers/wcs/platform/manage-api-keys?utm_source=chatgpt.com) · _Weaviate_  
To delete an API key, follow these steps: Click the Details button. Click the API keys button. Click the trash-can icon to delete the API key. Retrieve your API keys To retrieve your API keys, follow these steps: Open the WCD console and find the details panel for your cluster. Click the API keys button. Copy the key and store it in a safe ...
>



**注意事项**：

+ **安全性**：请妥善保管您的 API 密钥，避免泄露。建议将密钥存储在环境变量中，而非直接在代码中硬编码。 

> [Weaviate Api Key Management - Restackio](https://www.restack.io/p/weaviate-answer-api-key-management-cat-ai?utm_source=chatgpt.com) · _restack.io_ · 2025/1/7  
To effectively manage your Weaviate API keys, it is essential to follow best practices that ensure security and efficiency. Here are some key strategies: Secure Storage. Environment Variables: Store your API keys in environment variables instead of hardcoding them in your application. This minimizes the risk of exposure in your source code.  
e in your source code.
>

+ **定期更新**：定期更换 API 密钥，以提高安全性。
+ **权限控制**：根据用户角色，分配最小必要权限，遵循最小权限原则。

通过上述步骤，您可以成功申请并管理 Weaviate 的 API 密钥，确保对数据库的安全访问。

