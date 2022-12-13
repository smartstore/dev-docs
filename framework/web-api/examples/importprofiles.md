# ImportProfiles

### Upload import files and start to import

```
POST http://localhost:59318/odata/v1/ImportProfiles(16)/SaveFiles

Content-Type: text/plain; charset=utf-8
Content-Disposition: form-data; name=deleteFiles
True

Content-Type: text/plain; charset=utf-8
Content-Disposition: form-data; name=startImport
True

Content-Type: application/octet-stream
Content-Disposition: form-data; name="my-file-1"; filename="produktcsvexport.csv"
<Binary data for produktcsvexport.csv here (length 6645 bytes)…>
```
