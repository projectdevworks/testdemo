**edd-amazon-s3**

1. `web/app/plugins/edd-amazon-s3/class-edd-amazon-s3.php`

- modified `get_s3_url()` function.
- Added header
  - `'ResponseContentType'=> 'binary/octet-stream',`
  - `'ResponseContentDisposition' => 'attachment'`
- purpose : force donwload links to proceed downloading instead of playing or previewing files


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

2. `web/app/plugins/edd-amazon-s3/vendor/aws/aws-sdk-php/src/S3/S3Client.php`

- modified `getObjectUrl()` function.
- Added header
  - `'ResponseContentType'=> 'binary/octet-stream'`
  - `'ResponseContentDisposition' => 'attachment'`
- purpose : force donwload links to proceed download instead of playing or previewing files

```
public function getObjectUrl($bucket, $key)
{
	$command = $this->getCommand('GetObject', [
	    'Bucket' => $bucket,
	    'Key'    => $key,
	    'ResponseContentType'=> 'binary/octet-stream',
	    'ResponseContentDisposition' => 'attachment' 
	]);

	return (string) \Aws\serialize($command)->getUri();
}

```
