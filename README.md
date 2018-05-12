# Push-Notification
Push Notification For Android and IOS

**Usage**
```send("deviceToken", "message");```
```sentToIOS("deviceToken","message");```

*Firebase.php*
```php
<?php

define('FIREBASE_API_KEY_USER', 'YOUR_API_KEY');

class Firebase
{

// sending push message to single user by firebase reg id
    public function send($to, $message)
    {
        $fields = array(
            'to' => $to,
            'data' => $message,
        );
        return $this->sendPushNotification($fields);
    }

// Sending message to a topic by topic name
    public function sendToTopic($to, $message)
    {
        $fields = array(
            'to' => '/topics/' . $to,
            'data' => $message,
        );
        return $this->sendPushNotification($fields);
    }

// sending push message to multiple users by firebase registration ids
    public function sendMultiple($registration_ids, $message)
    {
        $fields = array(
            'to' => $registration_ids,
            'data' => $message,
        );

        return $this->sendPushNotification($fields);
    }

// function makes curl request to firebase servers
    private function sendPushNotification($fields)
    {
// Set POST variables
        $url = 'https://fcm.googleapis.com/fcm/send';
        $headers[] = 'Content-Type: application/json';
        $headers = array(
            'Authorization: key=' . FIREBASE_API_KEY_USER,
            'Content-Type: application/json'
        );
// Open connection
        $ch = curl_init();

// Set the url, number of POST vars, POST data
        curl_setopt($ch, CURLOPT_URL, $url);

        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "POST");
// Disabling SSL Certificate support temporarly
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);

        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($fields));

// Execute post
        $result = curl_exec($ch);
        if ($result === FALSE) {
            die('Curl failed: ' . curl_error($ch));
        }
        
        curl_close($ch);

        return $result;
    }

    function sendToIOS($token, $message)
    {
        try {
            if ($message == null || $token == null || empty($token)) {
                return "";
            }
            $json = array();
           
            $apnsHost = 'gateway.sandbox.push.apple.com';
            $apnsCert = 'pushcert.pem';
            $apnsPort = 2195;
            $streamContext = stream_context_create();
            stream_context_set_option($streamContext, 'ssl', 'local_cert', $apnsCert);
            $apns = stream_socket_client('ssl://' . $apnsHost . ':' . $apnsPort, $error, $errorString, 2, STREAM_CLIENT_CONNECT, $streamContext);
            $payload['aps'] = array('alert' => $message, 'badge' => 1, 'sound' => 'default');
            $output = json_encode($payload);
            $token = pack('H*', $token);
            $apnsMessage = chr(0) . chr(0) . chr(32) . $token . chr(0) . chr(strlen($output)) . $output;
           
            if (!$result) {
                $json = array("multicast_id"=>1,"success"=>1,"failure"=>0,"canonical_ids"=>0,"results"=>array("message_id"=>"5e6721f"));
            } else {
                $json = array("multicast_id"=>1,"success"=>1,"failure"=>1,"canonical_ids"=>0,"results"=>array("message_id"=>"5e6721f"));
            }

            fclose($apns);
            //echo $errorString;
        } catch (Exception $e) {
            //echo $e->getMessage();
        }
        return json_encode($json);
    }

}
```
