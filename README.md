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

**edd-fes**

1. `web/app/plugins/edd-fes/classes/class-emails.php`

- modified `send_email()` function.
- added BCC parameter
- purpose : added BCC for request quotation contributor form


```
public function send_email( $to, $from_name, $from_email, $subject, $message, $type, $id, $args = array(),  $bccEmail='' ) {

	if ( ! EDD_FES()->emails->should_send( $args ) ) {
		return false;
	}

	// start building the email
	$emails               = new EDD_Emails;
	$message_to_send      = $this->email_tags( $id, $message, $type );
	$message_to_send      = apply_filters( 'fes_send_mail_message', $message_to_send, $to, $from_name, $from_email, $subject, $message, $type, $id, $args );
	$emails->from_name    = $from_name;
	$emails->from_address = $from_email;
	$emails->heading      = $subject;
	
	
	// checks if bcc is provided and update email header 
	if( $bccEmail != '' ) {
		$headers = $emails->get_headers();
		$headers .= 'Bcc: ' . $bccEmail ; 
		$emails->__set( 'headers', $headers );
	}

	$emails->send( $to, $subject, $message_to_send );
}
```
