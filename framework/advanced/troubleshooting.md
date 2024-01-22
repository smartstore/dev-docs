# 🐣 Troubleshooting

Sometimes Smartstore's behavior can go completely out of control and become unresponsive. The following solutions can help identify and resolve a variety of issues.

## Extremely long text and MSSQL

`UseSequentialDbDataReader` is a new database system preference located in _appsettings.json_. When set to `true`, it can solve massive performance problems with databases overloaded with very long text entities. If you encounter texts larger than 300MB, MSSQL will crash. Set `UseSequentialDbDataReader` to `true` and Smartstore should run more smoothly.

{% hint style="info" %}
Postgre and MySQL are not affected by the large text problem.
{% endhint %}

## Offload embedded images

This function converts Base64 images to actual images in combination with an enabled `UseSequentialDbDataReader`. To accomplish this, it does the following:

1. Scans long texts for Base64 encoded images.
2. Extracts the images.
3. Places the images in the `MediaManager`.
4. Replaces the Base64 in the long text with a link to the image.

There is no UI for this, it's invoked via the URL: _/admin/maintenance/offloadembeddedimages/_ and requires admin access. This process can take a long time and use a lot of CPU. Therefore, the default setting is to process a maximum of `200` images per cycle.

If you want to process more images at once, avoiding to repeat the task multiple times, add the take URL-Parameter: _?take=\<MaxNumOfImagesToProcess>_

{% hint style="warning" %}
If you get a timeout error, you'll need to reduce the number of images to process.
{% endhint %}

When all embedded images have been successfully offloaded, set the `UseSequentialDbDataReader` setting back to `false`.
