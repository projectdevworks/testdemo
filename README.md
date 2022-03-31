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

2. `web/app/plugins/edd-fes/assets/js/admin.js`

- line : 120 - added a new onChange function for vendor-set-as-featured
- handle admin action to set Featured contributor

```
$('.vendor-set-as-featured').on('change',function(){
	var vendorId = $(this).data('vendor');
	var nstatus = $(this).data('nstatus');
	var status = $(this).prop('checked');

	var message_end='';
	if (status){
		message_end = fes_admin.vendor_status_change_message_end;
	}

	swal({
		title: fes_admin.vendor_status_change_title,
		text: fes_admin.vendor_status_change_message_start + nstatus + message_end,
		type: "warning",
		showCancelButton: true,
		confirmButtonColor: '#DD6B55',
		confirmButtonText: fes_admin.vendor_status_change_yes,
		cancelButtonText: fes_admin.vendor_status_change_no,
		closeOnConfirm: false,
		html: true,
		closeOnCancel: true
	},

	function( confirm ) {
		if (confirm){
			var opts = {
				lines: 13, // The number of lines to draw
				length: 11, // The length of each line
				width: 5, // The line thickness
				radius: 17, // The radius of the inner circle
				corners: 1, // Corner roundness (0..1)
				rotate: 0, // The rotation offset
				color: '#FFF', // #rgb or #rrggbb
				speed: 1, // Rounds per second
				trail: 60, // Afterglow percentage
				shadow: false, // Whether to render a shadow
				hwaccel: false, // Whether to use hardware acceleration
				className: 'fes_spinner', // The CSS class to assign to the spinner
				zIndex: 2e9, // The z-index (defaults to 2000000000)
				top: 'auto', // Top position relative to parent in px
				left: 'auto' // Left position relative to parent in px
			};

			var target = document.createElement("div");
			document.body.appendChild(target);
			if ( fes_admin.loading_icon != "" ){
				var overlay = fesSpinner({
					text: fes_admin.loadingtext,
					icon: fes_admin.loading_icon
				});
			} else {
				var spinner = new Spinner(opts).spin(target);
				var overlay = fesSpinner({
					text: fes_admin.loadingtext,
					spinner: spinner
				});
			}
			$.post( fes_admin.ajaxurl, { action:'fes_set_vendor_as_featured',id:vendorId,status:status}, function (response) {

			let res = JSON.parse(response)
				if (res.success) {
					overlay.hide();
					swal({ 
						title: res.title,
						text: res.message,
						html: true,
						type: "success"
					},
					function(){window.location.reload(true);
					} );     
				} else {
					overlay.hide();
					// show error overlay
					swal({
						title: res.title,
						text: res.message,
						html: true,
						type: "error"
					});
				}
			})
			.fail( function(xhr, textStatus, errorThrown) {
				var title = '';
				var message = '';
				title = fes_form.ajaxerrortitle;
				message = $(xhr.responseText).text();
				console.log( message );
				message = message.substring(0, message.indexOf("Call Stack"));
				overlay.hide(); // hide loading overlay
				// show error overlay
				swal({ 
					title: title,
					text: message,
					html: true,
					type: "error"
				}); 
			});
		}
	});

});

```

3. `web/app/plugins/edd-fes/classes/admin/vendors/vendors.php`

- line : 264 -added setFeatured admin

```
$admin_actions['setFeatured'] = array(
	'action' => 'setFeatured',
	'name'   => __( 'Feature', 'edd_fes' ),
	'label'  => __( 'Set contributor as featured', 'edd_fes' ),
	'url'    => '#',
);
```
- line : 301 modified foreach block to render a box for set contributor as featured

```
foreach ( $admin_actions as $action ) {
	if($action['action'] == 'setFeatured') {
		$user_meta = get_user_meta ( $user_id );
		if($user_meta['featured'][0]){
			$data .=   sprintF('<br><br><input type="checkbox" class="vendor-set-as-featured" data-vendor="%d"  data-nstatus="remove this contributor from being featured" checked /> Set as Featured Contributor',(int) esc_attr( $user_id ) );
		}
		else{
			$data .=   sprintF('<br><br><input type="checkbox" class="vendor-set-as-featured" data-vendor="%d"  data-nstatus="%s"/> Set as Featured Contributor',(int) esc_attr( $user_id ),esc_attr( $action['name'] ));
		}
	}
	else{
		$data  .= sprintf( '<a class="button tips vendor-change-status" data-vendor="%d" data-status="%s" data-nstatus="%s" href="%s" data-tip="%s">%s</a>', (int) esc_attr( $user_id ), esc_attr( $action['action'] ), esc_attr( $action['name'] ), esc_url( $action['url'] ), esc_attr( $action['name'] ), esc_attr( $action['label'] ) );
	}
}
```
- line : 1242 - added add_action fes_set_vendor_as_featured to add or update user meta

```
/**
 * FES set vendor as featured.
 *
 * set a vendor as featured.
 *
 * @param int    $id     Id of the user whose vendor account status is being changed.
 * @param string $status featured Status to change vendor to.
 *
 * @return mixed If output is requested, a JSON encoded array with message.
 */

 function fes_set_vendor_as_featured() {

	$output = array();

	if ( ! EDD_FES()->vendors->user_is_admin() ) {
		if ( $output ) {
			$output                = array();
			$output['title']       = __( 'Error!', 'edd_fes' );
			$output['message']     = __( 'You don\'t have the permissions to do that!', 'edd_fes' );
			$output['redirect_to'] = '#';
			$output['success']     = false;
			if ( fes_is_ajax_request() ) {
				echo json_encode( $output );
				exit;
			} else {
				return $output;
			}
		} else {
			return;
		}
	}


	update_user_meta( intval($_REQUEST['id']) , "featured", $_REQUEST['status'] === 'true' ? true : false);

	$output['success']     = true;
	$output['id'] = intval($_REQUEST['id']);
	$output['isFeatured'] = $_REQUEST['status'] === 'true' ? true : false;
	$output['title'] = __( 'Success!', 'edd_fes' );
	$output['message']     = __( 'Successfully updated Contributor!', 'edd_fes' );
	

	if ( fes_is_ajax_request() ) {
		echo json_encode( $output );
		exit;
	} else {
		return $output;
	}
	
}


add_action( 'wp_ajax_fes_set_vendor_as_featured', 'fes_set_vendor_as_featured', 10, 3 );
```
