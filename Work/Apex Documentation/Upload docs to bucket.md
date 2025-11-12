```curl
curl --location 'http://10.115.23.144:9097/manual/upload' \
--form 'user="vamsi"' \
--form 'storage_path="docs/"' \
--form 'file_name="curl-docs.zip"' \
--form 'uploadedBy="vamsi"' \
--form 'bucket="gw-prod-in-apex-static-bucket"' \
--form 'file=@"/Users/vamsik/Desktop/curl-docs.zip"'

curl --location 'http://10.115.23.144:9097/manual/upload' \
--form 'user="vamsi"' \
--form 'storage_path="docs/"' \
--form 'file_name="python-sdk-docs.zip"' \
--form 'uploadedBy="vamsi"' \
--form 'bucket="gw-prod-in-apex-static-bucket"' \
--form 'file=@"/Users/vamsik/Desktop/python-sdk-docs.zip"'
```
