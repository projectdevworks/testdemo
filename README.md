
**edd-amazon-s3**

1. `web/app/plugins/edd-amazon-s3/class-edd-amazon-s3.php`

- modified `get_s3_url()` function.
- Added header
  - `'ResponseContentDisposition' => 'attachment'`
- purpose : force donwload links to proceed downloadind instead of playing or previewing files

```
if ( class_exists( '\\Aws\\S3\\S3MultiRegionClient' ) ) {
	$object_command = $this->s3->getCommand( 'GetObject', array(
		'Bucket' => $bucket,
		'Key'    => $filename,
		'ResponseContentType'=> 'binary/octet-stream',
		'ResponseContentDisposition' => 'attachment'
	) );

	$request = $this->s3->createPresignedRequest( $object_command, '+' . $expires . ' minutes' );

	$url = (string) $request->getUri();

	return $url;
} else {
	return $this->s3->getObjectUrl( $bucket, $filename, '+10 minutes' );
}
```



