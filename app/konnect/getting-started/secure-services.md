---
title: Secure your Services Using Authentication
no_version: true
---
***TO DO: Rework for Konnect Cloud***

In this topic, you’ll learn about API Gateway authentication, set up the Key Authentication plugin, and add a consumer.  

If you are following the getting started workflow, make sure you have completed [Improve Performance with Proxy Caching](/konnect/getting-started/improve-performance) before moving on.

## What is Authentication?

API gateway authentication is an important way to control the data that is allowed to be transmitted using your APIs. Basically, it checks that a particular consumer has permission to access the API, using a predefined set of credentials.

{{site.base_gateway}} has a library of plugins that provide simple ways to implement the best known and most widely used [methods of API gateway authentication](/hub/#authentication). Here are some of the commonly used ones:

* Basic Authentication
* Key Authentication
* OAuth 2.0 Authentication
* LDAP Authentication Advanced
* OpenID Connect

Authentication plugins can be configured to apply to service entities within the {{site.base_gateway}}. In turn, service entities are mapped one-to-one with the upstream services they represent, essentially meaning that the authentication plugins apply directly to those upstream services.

## Why use API Gateway Authentication?

With authentication turned on, {{site.base_gateway}} won’t proxy requests unless the client successfully authenticates first. This means that the upstream (API) doesn’t need to authenticate client requests, and it doesn’t waste critical resources validating credentials.

{{site.base_gateway}} has visibility into all authentication attempts, successful, failed, and so on, which provides the ability to catalog and dashboard those events to prove the right controls are in place, and to achieve compliance. Authentication also gives you an opportunity to determine how a failed request is handled. This might mean simply blocking the request and returning an error code, or in some circumstances, you might still want to provide limited access.

In this example, you’re going to enable the **Key Authentication plugin**. API key authentication is one of the most popular ways to conduct API authentication and can be implemented to create and delete access keys as required.

For more information, see [What is API Gateway Authentication?](https://konghq.com/learning-center/api-gateway/api-gateway-authentication/).

## Set up the Key Authentication Plugin

{% navtabs %}
{% navtab Using Kong Manager %}

1. Access your Kong Manager instance and your **default** workspace.
2. Go to the **Routes** page and select the **mocking** route you created.
3. Click **View**.
4. On the Scroll down and select the **Plugins** tab, then click **Add a Plugin**.
5. In the Authentication section, find the **Key Authentication** plugin and click **Enable**.
6. In the **Create new key-auth plugin** dialog, the plugin fields are automatically scoped to the route because the plugin is selected from the mocking Routes page.

    For this example, this means that you can use all of the default values.

7. Click **Create**.

    Once the plugin is enabled on the route, **key-auth** displays under the Plugins section on the route’s overview page.

Now, if you try to access the route without providing an API key, the request will fail, and you’ll see the message `"No API key found in request".`

Before Kong proxies requests for this route, it needs an API key. For this example, since you installed the Key Authentication plugin, you need to create a consumer with an associated key first.
{% endnavtab %}
{% navtab Using decK (YAML) %}
1. Under the `mocking` route in the `routes` section of the `kong.yaml` file,
add a plugin section and enable the `key-auth` plugin:

    ``` yaml
    plugins:
    - name: key-auth
    ```

    Your file should now look like this:

    ``` yaml
    _format_version: "1.1"
    services:
    - host: mockbin.org
      name: example_service
      port: 80
      protocol: http
      routes:
      - name: mocking
        paths:
        - /mock
        strip_path: true
        plugins:
        - name: key-auth
    plugins:
    - name: rate-limiting
      config:
        minute: 5
        policy: local
    - name: proxy-cache
      config:
        content_type:
        - "application/json; charset=utf-8"
        cache_ttl: 30
        strategy: memory
    ```

2. Sync the configuration:

    ``` bash
    deck sync
    ```

Now, if you try to access the route at `http://<admin-hostname>:8000/mock`
without providing an API key, the request will fail, and you’ll see the message
`"No API key found in request".`

Before Kong proxies requests for this route, it needs an API key. For this
example, since you installed the Key Authentication plugin, you need to create
a consumer with an associated key first.

{% endnavtab %}
{% endnavtabs %}


## Set up Consumers and Credentials

{% navtabs %}
{% navtab Using Kong Manager %}

1. In Kong Manager, go to **API Gateway** > **Consumers**.
2. Click **New Consumer**.
3. Enter the **Username** and **Custom ID**. For this example, use `consumer` for each field.
4. Click **Create**.
5. On the Consumers page, find your new consumer and click **View**.
6. Scroll down the page and click the **Credentials** tab.
7. Click **New Key Auth Credential**.
8. Set the key to **apikey** and click **Create**.

  The new Key Authentication ID displays on the **Consumers** page under the **Credentials** tab.
{% endnavtab %}
{% navtab Using decK (YAML) %}
1. Add a `consumers` section to your `kong.yaml` file and create a new consumer:

    ``` yaml
    consumers:
    - custom_id: consumer
      username: consumer
      keyauth_credentials:
      - key: apikey
    ```

    Your file should now look like this:

    ``` yaml
    _format_version: "1.1"
    services:
    - host: mockbin.org
      name: example_service
      port: 80
      protocol: http
      routes:
      - name: mocking
        paths:
        - /mock
        strip_path: true
        plugins:
        - name: key-auth
    consumers:
    - custom_id: consumer
      username: consumer
      keyauth_credentials:
      - key: apikey
    plugins:
    - name: rate-limiting
      config:
        minute: 5
        policy: local
    - name: proxy-cache
      config:
        content_type:
        - "application/json; charset=utf-8"
        cache_ttl: 30
        strategy: memory
    ```

2. Sync the configuration:

    ``` bash
    deck sync
    ```

You now have a consumer with an API key provisioned to access the route.

{% endnavtab %}
{% endnavtabs %}


## Validate Key Authentication

To validate the Key Authentication plugin, access your route through your browser by appending `?apikey=apikey` to the url:
```
http://<admin-hostname>:8000/mock/?apikey=apikey
```


## (Optional) Disable the plugin
If you are following this getting started guide topic by topic, you will need to use this API key in any requests going forward. If you don’t want to keep specifying the key, disable the plugin before moving on.

{% navtabs %}
{% navtab Using Kong Manager %}
1. Go to the Plugins page and click on **View** for the key-auth row.
2. Use the toggle at the top of the page to switch the plugin from **Enabled** to **Disabled**.
{% endnavtab %}
{% navtab Using decK (YAML) %}

1. Disable the key-auth plugin in the `kong.yaml` file by setting
`enabled` to `false`:

    ``` yaml
    plugins:
    - name: key-auth
      enabled: false
    ```

2. Sync the configuration:

    ``` bash
    $ deck sync
    ```

{% endnavtab %}
{% endnavtabs %}

## Summary and next steps

In this topic, you:

* Enabled the Key Authentication plugin.
* Created a new consumer named `consumer`.
* Gave the consumer an API key of `apikey` so that it could access the `/mock` route with authentication.

Next, you’ll learn about [load balancing upstream services using targets](/konnect/getting-started/load-balancing).
