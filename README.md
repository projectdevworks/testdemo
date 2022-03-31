# testdemo
## testdemo
###### testdemo
<pre>
if(code == to testr) {
  <strong>console.log("all is well")</strong>
  **test**
}
</pre>

```php

if ( class_exists( '\\Aws\\S3\\S3MultiRegionClient' ) ) {
	$object_command = $this->s3->getCommand( 'GetObject', array(
		'Bucket' => $bucket,
		'Key'    => $filename,
```
		**'ResponseContentType'=> 'binary/octet-stream',**
		**'ResponseContentDisposition' => 'attachment'**
```php
	) );

	$request = $this->s3->createPresignedRequest( $object_command, '+' . $expires . ' minutes' );

	$url = (string) $request->getUri();

	return $url;
} else {
	return $this->s3->getObjectUrl( $bucket, $filename, '+10 minutes' );
}
```



