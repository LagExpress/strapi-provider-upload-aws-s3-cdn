# strapi-provider-upload-s3-cdn

This is bassically a carbon copy of the official [@strapi/provider-upload-aws-s3](https://www.npmjs.com/package/@strapi/provider-upload-aws-s3) package, with just a small addition to the `upload` method to support the `cdn` option.

## Resources

- [LICENSE](LICENSE)

## Links

- [Strapi website](https://strapi.io/)
- [Strapi documentation](https://docs.strapi.io)
- [Strapi community on Discord](https://discord.strapi.io)
- [Strapi news on Twitter](https://twitter.com/strapijs)

## Installation

```bash
# using yarn
yarn add strapi-provider-upload-s3-cdn

# using npm
npm install strapi-provider-upload-s3-cdn
```

## Configurations

Your configuration is passed down to the provider. (e.g: `new AWS.S3(config)`). You can see the complete list of options [here](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#constructor-property)

See the [using a provider](https://docs.strapi.io/developer-docs/latest/plugins/upload.html#using-a-provider) documentation for information on installing and using a provider. And see the [environment variables](https://docs.strapi.io/developer-docs/latest/setup-deployment-guides/configurations/optional/environment.html#environment-variables) for setting and using environment variables in your configs.

### Provider Configuration

`./config/plugins.js`

```js
module.exports = ({ env }) => ({
  // ...
  upload: {
    config: {
      provider: "aws-s3",
      providerOptions: {
        accessKeyId: env("AWS_ACCESS_KEY_ID"),
        secretAccessKey: env("AWS_ACCESS_SECRET"),
        // endpoint: env('AWS_ENDPOINT'), // if you are using an s3 compatible service like minio or backblaze uncomment this line and comment the next one
        region: env("AWS_REGION"),
        cdn: env("AWS_CDN"), // if you want to use a cdn, you can set the url here also if you decide to not use one just leave it empty
        params: {
          Bucket: env("AWS_BUCKET"),
        },
      },
    },
  },
  // ...
});
```

### Security Middleware Configuration

Due to the default settings in the Strapi Security Middleware you will need to modify the `contentSecurityPolicy` settings to properly see thumbnail previews in the Media Library. You should replace `strapi::security` string with the object bellow instead as explained in the [middleware configuration](https://docs.strapi.io/developer-docs/latest/setup-deployment-guides/configurations/required/middlewares.html#loading-order) documentation.

`./config/middlewares.js`

```js
module.exports = [
  // ...
  {
    name: "strapi::security",
    config: {
      contentSecurityPolicy: {
        useDefaults: true,
        directives: {
          "connect-src": ["'self'", "https:"],
          "img-src": [
            "'self'",
            "data:",
            "blob:",
            "yourBucketName.s3.yourRegion.amazonaws.com",
          ],
          "media-src": [
            "'self'",
            "data:",
            "blob:",
            "yourBucketName.s3.yourRegion.amazonaws.com",
          ],
          upgradeInsecureRequests: null,
        },
      },
    },
  },
  // ...
];
```

## Required AWS Policy Actions

These are the minimum amount of permissions needed for this provider to work.

```json
"Action": [
  "s3:PutObject",
  "s3:GetObject",
  "s3:ListBucket",
  "s3:DeleteObject",
  "s3:PutObjectAcl"
],
```
