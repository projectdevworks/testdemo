# testdemo
## testdemo
###### testdemo
<pre>
if(code == to testr) {
  <strong>console.log("all is well")</strong>
  **test**
}
</pre>


<pre>
	if ( class_exists( '\\Aws\\S3\\S3MultiRegionClient' ) ) {
		$object_command = $this->s3->getCommand( 'GetObject', array(
			'Bucket' => $bucket,
			'Key'    => $filename,
			<strong><em>'ResponseContentType'=> 'binary/octet-stream',</em></strong>
			<strong><em>'ResponseContentDisposition' => 'attachment'</em></strong>
		) );

		$request = $this->s3->createPresignedRequest( $object_command, '+' . $expires . ' minutes' );

		$url = (string) $request->getUri();

		return $url;
	} else {
		return $this->s3->getObjectUrl( $bucket, $filename, '+10 minutes' );
	}
</pre>



