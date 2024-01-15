# quick-samples-rcubedev

### Whatsapp Cloud Messaging API Integration;
**OTP Auth Messaging**
```php
function sendWhatsAppMsg($sendTo, $action, $params)
    {
        $template = getWATemplate($action, $params);
        // dd($template);

        $lang = $template['language'];
        $temp_name = $template['template_name'];
        $temp_id = $template['template_id'];

        $curl = curl_init();
        
        // ToDo: Update API Data Dynamic;
        $apiVersion = 'v17.0';
        $apiSenderId = '154391567754698';
        $apiAuthKey = 'EAAPRPZAZCDgBcBO3z8hD7nNZBaJjM7iqPv8ISd41rC2crtCblSDiZC9lfHuS7AKv4oePblPibLZAqsW6ey9Hinw7ilgNVMknnfEkbpiBmqEpI7ohGK8OmeHs7zCKYCIXTA0PUVAZCcZChS2j0ZBNn2y40jPTZB4i2EBiFo5TkX42UDO5Hj4vlo1yAbDEglz1dgq27g3afHyZBdZC9ogYZCNzix8ZD';

        $endpoint = 'https://graph.facebook.com/'.$apiVersion.'/'.$apiSenderId.'/messages';

        $headers = array(
            'Content-Type: application/json',
            'Authorization: Bearer '.$apiAuthKey,
        );

        $body =
        '{
            "messaging_product": "whatsapp",
            "to": "918697034671",
            "type": "template",
            "template": {
                "name": "dummy_otp_request",
                "language": {
                    "code": "en",
                    "policy": "deterministic"
                },
                "components": [
                    {
                        "type": "body",
                        "parameters": [
                            {
                                "type": "text",
                                "text": "08697"
                            }
                        ]
                    },
                    {
                        "type": "button",
                        "sub_type": "url",
                        "index": 0,
                        "parameters": [
                            {
                                "type": "text",
                                "text": "08697"
                            }
                        ]
                    }
                ]
            },

        }';
        
        curl_setopt($curl, CURLOPT_URL, $endpoint);
        curl_setopt($curl, CURLOPT_POST, true);
        curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
        curl_setopt($curl, CURLOPT_POSTFIELDS, $body);
        curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);

        $res = curl_exec($curl);
        if(curl_errno($curl)) {
            $error = curl_error($curl);
            dd($error);
            return $error;
        }

        if(!isset($error))
        {
            $response = json_decode($res, true);
            dd($response);
        }
    }
```

### Random String Generation
```php
if (!function_exists('getRandomString')) {
    function getRandomString($n)
    {
        $characters = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ';
        $randomString = '';

        for ($i = 0; $i < $n; $i++) {
            $index = rand(0, strlen($characters) - 1);
            $randomString .= $characters[$index];
        }

        return $randomString;
    }
}

if (!function_exists('getRandomStringSmall')) {
    function getRandomStringSmall($n)
    {
        $characters = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
        $randomString = '';

        for ($i = 0; $i < $n; $i++) {
            $index = rand(0, strlen($characters) - 1);
            $randomString .= $characters[$index];
        }

        return $randomString;
    }
}
```

### Patch Log
```php
if (!function_exists('patchLog')) {
    function patchLog(
        $refId = null,
        $refName = null,
        $initiator = null,
        $module = null,
        $action = null,
        $beforeValue = null,
        $afterValue = null,
        $remarks = null,
        $logDevice = 'Web'
    ) {
        Log::create([
            'ref_id' => $refId,
            'ref_name' => $refName,
            'initiator' => session('associate_id'),
            'module' => $module,
            'action' => $action,
            'before_value' => $beforeValue,
            'after_value' => $afterValue,
            'remarks' => $remarks,
            'log_ip' => request()->ip(),
            'log_device' => $logDevice,
            'log_agent' => request()->header('User-Agent'),
        ]);
    }
}
```

### Working with Laravel Excel Package;
Laravel Excel - [Docs](https://docs.laravel-excel.com/3.1/getting-started/)

```php
<?php

namespace App\Imports;

use App\Models\Pincode;
use App\Models\RewardCategory;
use App\Models\RewardPartner;
use Maatwebsite\Excel\Concerns\ToModel;
use Maatwebsite\Excel\Concerns\WithStartRow;

class RewardPartnersImport implements ToModel, WithStartRow
{
    /**
    * @param array $row
    *
    * @return \Illuminate\Database\Eloquent\Model|null
    */
    private $ucode;
    private $rows = 0;

    public function __construct() {
        $this->ucode = 'RRP'.date("y");
    }

    public function startRow(): int
    {
        return 2;
    }

    public function model(array $row)
    {
        // Model Insertion;
        ++$this->rows;

        // UUID->
        $rp = RewardPartner::orderBy('id', 'DESC')->limit(1)->first();
        if(empty($rp)) {
            $new_number = "0001";
            $uuid = $this->ucode.$new_number;
        } else {
            $rp_uuid = $rp['uuid'];
            $ain_num = substr($rp_uuid, 5);
            // $ain_num = explode($curr_ucode, $rp_uuid);
            $new_number = str_pad($ain_num+1,4,"0",STR_PAD_LEFT);
            $uuid = $this->ucode.$new_number;
        }
        
        //Rows;
        $pincode = $row[0];
        $category = $row[1];
        $brand_title = $row[2];
        $address = $row[3];
        $phone = $row[4];
        $phone2 = $row[5];
        $email = $row[6];
        $website = $row[8];


        // Find Pincode_ID;
        $pc = Pincode::where('pincode', $pincode)->first();
        
        // Find Reward_Category ID;
        $cat= RewardCategory::where('title', $category)->first();
        if($cat) {
            $cat_id = $cat->id; 
        } else {
            $new_cat = RewardCategory::create([
                'title' => $category,
                'parent' => null,
                'published' => 'Yes',
            ]);

            $cat_id = $new_cat->id;
        }

        // Insert Record;
        return new RewardPartner([
            'uuid' => $uuid,
            'reward_category' => $cat_id,
            'pincode' => $pc->id ?? 0,
            'brand_name' => $brand_title,
            'phone_number' => $phone,
            'contact_number' => $phone2,
            'address' => $address,
            'logo' => null,
            'email' => $email,
            'website' => $website,
            'gstin' => null,
            'published' => 'Yes',
            'created_by' => session('associate_id'),
            'created_at' => now(),
            'tagged_by' => null,
            'tagged_on' => null,
            'onboarded_by' => null,
            'onboarded_on' => null,
            'onboard_type' => null,
            'is_sponsored' => 'No'             
        ]);
    }

    public function getRowCount(): int
    {
        return $this->rows;
    }
}
```

### Custom Filtering using Datatables;

```html
<div id="typeFilter-cont" class="col-md-3 mb-3">
    <label for="" class="form-label">Filter Partner Type</label>
    <select name="typeFilter" id="typeFilter" class="select2 form-select form-control border">
        <option value="">All Partners</option>
        // Types for Filtering;
        @foreach ($partnerTypes as $type)
            <option value="{{ $type->title }}">{{ $type->title }}</option>
        @endforeach
    </select>
</div>
```

```js
$(document).ready(function() {
        $('#myDataTable').dataTable({
            "searching": true
        });

        var table = $('#myDataTable').DataTable();
        
        var typeIndex = 0;
        $("#myDataTable th").each(function (i) {
            console.log($($(this)).html());
            if ($($(this)).html() == "Partner Type") {
                typeIndex = i; return false;
            }
        });

        $.fn.dataTable.ext.search.push(
            function (settings, data, dataIndex) {
            var selectedItem = $('#typeFilter').val()
            var type = data[typeIndex];
            if (selectedItem === "" || type.includes(selectedItem)) {
                return true;
            }
            return false;
            }
        );


        $('#typeFilter').change(function (e) {
            table.draw();
        });

        table.draw();
});
```

## - Previous One

```php
if ($s == 'user_wallet_update') {
        global $wo, $sqlConnect;
        if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['price']) && isset($_POST['type'])) {
            $price = $_POST['price'];
            $type = $_POST['type'];

            // if ($wo['config']['affiliate_system'] == 1) {

            //     function getReferrerId($referrerId, $sqlConnect)
            //     {
            //         $query = "SELECT `referrer` FROM " . T_USERS . " WHERE `user_id` = '{$referrerId}'";
            //         $result = mysqli_query($sqlConnect, $query);
            //         if (mysqli_num_rows($result) > 0) {
            //             $data = mysqli_fetch_assoc($result);
            //             return $data['referrer'];
            //         } else {
            //             return null;
            //         }
            //     }

            //     $user_id = $wo['user']['user_id'];

            //     // Retrieve the current user's data
            //     $query1 = "SELECT * FROM " . T_USERS . "  WHERE `user_id` = '{$user_id}'";
            //     $sql1 = mysqli_query($sqlConnect, $query1);
            //     if (mysqli_num_rows($sql1)) {
            //         $data1 = mysqli_fetch_assoc($sql1);
            //     }

            //     $referrerId = $data1['referrer'];

            //     // RCD Mod Here: REQUIRED;
            //     if ($wo['config']['affiliate_request'] == 'all') {
            //         // Loop through the levels, starting from 1 up to 3
            //         // Loop through the levels until there are no more referrers
            //         $level = 1;
            //         while ($referrerId != 0 && $level<=5) {
            //             // Retrieve the data of the referrer
            //             $query = "SELECT * FROM " . T_USERS . " WHERE `user_id` = '{$referrerId}'";
            //             $result = mysqli_query($sqlConnect, $query);
            //             if (mysqli_num_rows($result) > 0) {
            //                 $referrerData = mysqli_fetch_assoc($result);

            //                 if ($referrerData['is_pro'] == '1') {
            //                     // Retrieve the commission percentage for the specific level
            //                     $query3 = "SELECT * FROM " . T_CONFIG . " WHERE `name` = 'user_level" . $level . "_per'";
            //                     $sql3 = mysqli_query($sqlConnect, $query3);
            //                     if (mysqli_num_rows($sql3) > 0) {
            //                         $levelData = mysqli_fetch_assoc($sql3);
            //                         $commissionPercentage = $levelData['value'];
            //                     }

            //                     // Retrieve the plan price for the referrer's pro type
            //                     $plan_price_query = "SELECT `price` FROM " . T_MANAGE_PRO . " WHERE `id` = '{$referrerData['pro_type']}'";
            //                     $result2 = mysqli_query($sqlConnect, $plan_price_query);

            //                     if ($result2 && mysqli_num_rows($result2) > 0) {
            //                         $row = mysqli_fetch_assoc($result2);
            //                         $plan_price = $row['price'];
            //                     }

            //                     if($plan_price < $price){
            //                         $commissionAmount = $plan_price * $commissionPercentage / 100;
            //                         // Update the referrer's wallet with the commission amount
            //                         $newWalletAmount = $referrerData['wallet'] + $commissionAmount;
            //                         $query = "UPDATE " . T_USERS . " SET `wallet` = '{$newWalletAmount}', `balance` = '{$newWalletAmount}' WHERE `user_id` = '{$referrerId}'";
            //                         mysqli_query($sqlConnect, $query);
            //                         }else{
            //                             $commissionAmount = $price * $commissionPercentage / 100;
            //                         // Update the referrer's wallet with the commission amount
            //                         $newWalletAmount = $referrerData['wallet'] + $commissionAmount;
            //                         $query = "UPDATE " . T_USERS . " SET `wallet` = '{$newWalletAmount}', `balance` = '{$newWalletAmount}' WHERE `user_id` = '{$referrerId}'";
            //                         mysqli_query($sqlConnect, $query);
            //                         }
            //                 }
            //                 else {
            //                     $query3 = "SELECT * FROM " . T_CONFIG . " WHERE `name` = 'free_affiliate_per'";
            //                     $sql3 = mysqli_query($sqlConnect, $query3);
            //                     if (mysqli_num_rows($sql3) > 0) {
            //                         $levelData = mysqli_fetch_assoc($sql3);
            //                         $commissionPercentage = $levelData['value'];
            //                     }
            //                     $commissionAmount = $price * $commissionPercentage / 100;
            //                     // Update the referrer's wallet with the commission amount
            //                     $newWalletAmount = $referrerData['wallet'] + $commissionAmount;
            //                     $query = "UPDATE " . T_USERS . " SET `wallet` = '{$newWalletAmount}', `balance` = '{$newWalletAmount}' WHERE `user_id` = '{$referrerId}'";
            //                     mysqli_query($sqlConnect, $query);
            //                     // Get the next referrer ID
                               
            //                 }

            //                 // Get the next referrer ID
            //                 $referrerId = getReferrerId($referrerId, $sqlConnect);
            //                 $level++; // Move to the next level
            //             } else {
            //                 break; // No referrer found, exit the loop
            //             }
            //         }
            //     }
            // }

            // If Referee is 6th Person in Level 1:
            if($wo['config']['rcd_fastStartActive'] == 1)
            {
                // 1. Check Referrer Plan:
                // 2. Check Referee Plan:
                // 3. Follow Conditions:
                // - Referrer will get (balance) only 50% of his plan;
                // - If Referee Plan is Lower, than referrer will get (balance) only 50% of buyers plan               
            }

            // Deduct the price from the user's wallet
            $newWalletAmount = $wo['user']['wallet'] - $price;
            // RCD Mod - commented on 07-01-24
            // $newBalanceAmount = $wo['user']['balance'] - $price;
            // $query5 = "UPDATE " . T_USERS . " SET `wallet` = '{$newWalletAmount}', `balance` = '{$newBalanceAmount}' WHERE `user_id` = '{$wo['user']['user_id']}'";
            $query5 = "UPDATE " . T_USERS . " SET `wallet` = '{$newWalletAmount}' WHERE `user_id` = '{$wo['user']['user_id']}'";
            $sql5 = mysqli_query($sqlConnect, $query5);
        }
    }
```
