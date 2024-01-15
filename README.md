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

## First One

```php
if ($s == 'pay' && $wo['loggedin'] === true) {
        
        // $data = array(
        //     'status' => 200,
        //     'message' => 'hitting here'
        // );
        // header("Content-type: application/json");
        // echo json_encode($data);
        // exit();

        $data  = array(
            'status' => 400
        );
        $price = 0;
        if (!empty($_GET['type']) && in_array($_GET['type'], array(
            'pro',
            'fund'
        )))
         {
            if ($_GET['type'] == 'pro') {
                $img             = "";
                $balance = false;
                if ($_GET['source'] = true) {
                    $balance = true;
                }
                if (!empty($_GET['pro_type']) && in_array($_GET['pro_type'], array_keys($wo["pro_packages"]))) {
                    $_GET['pro_type'] = Wo_Secure($_GET['pro_type']);

                    $img = $wo["pro_packages"][$_GET['pro_type']]['name'];

                    if ($balance == false) {

                        if ($wo["pro_packages"][$_GET['pro_type']]['price'] > $wo['user']['wallet'] && $wo["pro_packages"][$pro_type]['price'] > $datas['balance']) {

                            $data['message'] = "<a href='" . $wo['config']['site_url'] . "/wallet'>" . $wo["lang"]["please_top_up_wallet"] . "</a>";
                        } else {
                            
                            $price = $wo["pro_packages"][$_GET['pro_type']]['price'];
                        }
                    }
                } else {
                    $data['message'] = $error_icon . $wo['lang']['something_wrong'];
                }
            } 
            elseif ($_GET['type'] == 'fund') {
                if (!empty($_GET['price']) && is_numeric($_GET['price']) && $_GET['price'] > 0) {
                    if (!empty($_GET['fund_id']) && is_numeric($_GET['fund_id']) && $_GET['fund_id'] > 0) {
                        $fund_id = Wo_Secure($_GET['fund_id']);
                        $price   = Wo_Secure($_GET['price']);
                        $fund    = $db->where('id', $fund_id)->getOne(T_FUNDING);
                        if (empty($fund)) {
                            $data['message'] = $error_icon . $wo['lang']['fund_not_found'];
                        }
                    } else {
                        $data['message'] = $error_icon . $wo['lang']['something_wrong'];
                    }
                } else {
                    $data['message'] = $error_icon . $wo['lang']['amount_can_not_empty'];
                }
            }
            if (empty($data['message'])) {
                $balance = false;
                if ($_GET['source'] = true) {
                    $balance = true;
                }
                if ($_GET['type'] == 'pro') {
                    $is_pro = 0;
                    $stop   = 0;

                    // New code for point no 2 
                    $user_date = Wo_UserData($wo['user']['ref_user_id']);
                    $user_commission_ref = 0;

                    if (!empty($user_date)) {
                        if (!empty($user_date['user_level']) && empty($user_date['is_pro'])) {
                            if ($user_date['user_level'] == 1) {
                                $user_commission_ref = $wo['config']['free_affiliate_level1_per'];
                            } elseif ($user_date['user_level'] == 2) {
                                $user_commission_ref = $wo['config']['free_affiliate_level2_per'];
                            }
                        }

                        if (!empty($user_date['user_level']) && !empty($user_date['is_pro'])) {
                            if ($user_date['user_level'] == 1) {
                                $user_commission_ref = $wo['config']['pro_affiliate_level1_per'];
                            } elseif ($user_date['user_level'] == 2) {
                                $user_commission_ref = $wo['config']['pro_affiliate_level2_per'];
                            }
                        }
                    }
                    
                    // New code for point no 2 end

                    // $user   = Wo_UserData($wo['user']['user_id']);
                    // if ($user['is_pro'] == 1) {
                    //     $stop = 1;
                    //     if ($user['pro_type'] == 1) {
                    //         $time_ = time() - $star_package_duration;
                    //         if ($user['pro_time'] > $time_) {
                    //             $stop = 1;
                    //         }
                    //     } else if ($user['pro_type'] == 2) {
                    //         $time_ = time() - $hot_package_duration;
                    //         if ($user['pro_time'] > $time_) {
                    //             $stop = 1;
                    //         }
                    //     } else if ($user['pro_type'] == 3) {
                    //         $time_ = time() - $ultima_package_duration;
                    //         if ($user['pro_time'] > $time_) {
                    //             $stop = 1;
                    //         }
                    //     } else if ($user['pro_type'] == 4) {
                    //         if ($vip_package_duration > 0) {
                    //             $time_ = time() - $vip_package_duration;
                    //             if ($user['pro_time'] > $time_) {
                    //                 $stop = 1;
                    //             }
                    //         }
                    //     }
                    // }

                    if ($stop == 0) {
                        $pro_type        = $_GET['pro_type'];
                        $is_pro          = 1;
                    }
                    if ($stop == 0) {
                        $time = time();
                        if ($is_pro == 1) {
                            $update_array = array(
                                'is_pro' => 1,
                                'pro_time' => time(),
                                'pro_' => 1,
                                'pro_type' => $pro_type
                            );
                            if (in_array($pro_type, array_keys($wo['pro_packages'])) && $wo["pro_packages"][$pro_type]['verified_badge'] == 1) {
                                $update_array['verified'] = 1;
                            }
                            $mysqli             = Wo_UpdateUserData($wo['user']['user_id'], $update_array);
                            //$notes              = $wo['lang']['upgrade_to_pro'] . " " . $img . " : Wallet";
                            //$notes              = $img . " : Wallet";
                            //$notes              = str_replace('{text}', $img . " : Wallet", $wo['lang']['trans_upgrade_to_pro']);
                            if ($balance) {
                                $notes = json_encode([
                                    'pro_type' => $pro_type,
                                    'method_type' => 'balance'
                                ]);
                            } else {
                                $notes = json_encode([
                                    'pro_type' => $pro_type,
                                    'method_type' => 'wallet'
                                ]);
                            }
                            // $notes = json_encode([
                            //     'pro_type' => $pro_type,
                            //     'method_type' => 'wallet'
                            // ]);

                            $create_payment_log = mysqli_query($sqlConnect, "INSERT INTO " . T_PAYMENT_TRANSACTIONS . " (`userid`, `kind`, `amount`, `notes`) VALUES ({$wo['user']['user_id']}, 'PRO', {$price}, '{$notes}')");
                            $create_payment     = Wo_CreatePayment($pro_type,$wo['user']['user_id']);

                            
                            
                            if ($mysqli) {

                                // if ((!empty($_SESSION['ref']) || !empty($wo['user']['ref_user_id'])) && $wo['config']['affiliate_type'] == 1 && $wo['user']['referrer'] == 0) {
                                if ((!empty($_SESSION['ref']) || !empty($wo['user']['ref_user_id'])) && $wo['config']['affiliate_type'] == 1) {
                                    if (!empty($_SESSION['ref'])) {
                                        $ref_user_id = Wo_UserIdFromUsername($_SESSION['ref']);
                                    } elseif (!empty($wo['user']['ref_user_id'])) {
                                        $ref_user_id = $wo['user']['ref_user_id'];
                                    }
                                    if ($user_commission_ref > 0) {
                                        if (!empty($ref_user_id) && is_numeric($ref_user_id)) {
                                            $update_user = Wo_UpdateUserData($wo['user']['user_id'], array(
                                                'referrer' => $ref_user_id,
                                                'src' => 'Referrer'
                                            ));
                                            $ref_amount  = ($user_commission_ref * $price) / 100;
                                            if ($wo['config']['affiliate_level'] < 2) {
                                                $update_balance = Wo_UpdateBalance($ref_user_id, $ref_amount);
                                            }
                                            if (is_numeric($wo['config']['affiliate_level']) && $wo['config']['affiliate_level'] > 1) {
                                                AddNewRef($ref_user_id, $wo['user']['user_id'], $ref_amount, $price);
                                            }
                                            unset($_SESSION['ref']);
                                        }
                                    }

                                    // else if ($wo['config']['amount_ref'] > 0) {
                                    //     if (!empty($ref_user_id) && is_numeric($ref_user_id)) {
                                    //         $update_user = Wo_UpdateUserData($wo['user']['user_id'], array(
                                    //             'referrer' => $ref_user_id,
                                    //             'src' => 'Referrer'
                                    //         ));
                                    //         if($wo['config']['affiliate_level'] < 2) {
                                    //             $update_balance = Wo_UpdateBalance($ref_user_id, $wo['config']['amount_ref']);
                                    //         }
                                    //         if (is_numeric($wo['config']['affiliate_level']) && $wo['config']['affiliate_level'] > 1) {
                                    //             AddNewRef($ref_user_id, $wo['user']['user_id'], $wo['config']['amount_ref']);
                                    //         }
                                    //         unset($_SESSION['ref']);
                                    //     }
                                    // }

                                }

                                $points = 0;
                                if ($wo['config']['point_level_system'] == 1) {
                                    $points = $price * $dollar_to_point_cost;
                                }
                                if ($balance) {
                                    $wallet_amount  = ($wo["user"]['wallet'] - $price);
                                } else {
                                    $wallet_amount  = ($wo["user"]['balance'] - $price);
                                }
                                // $wallet_amount  = ($wo["user"]['wallet'] - $price);
                                $points_amount  = ($wo['config']['point_allow_withdrawal'] == 0) ? ($wo["user"]['points'] - $points) : $wo["user"]['points'];
                                if ($balance) {
                                    $query_one      = mysqli_query($sqlConnect, "UPDATE " . T_USERS . " SET `points` = '{$points_amount}', `balance` = '{$wallet_amount}' WHERE `user_id` = {$wo['user']['user_id']} ");
                                } else {
                                    $query_one      = mysqli_query($sqlConnect, "UPDATE " . T_USERS . " SET `points` = '{$points_amount}', `wallet` = '{$wallet_amount}' WHERE `user_id` = {$wo['user']['user_id']} ");
                                }
                                //  $query_one      = mysqli_query($sqlConnect, "UPDATE " . T_USERS . " SET `points` = '{$points_amount}', `wallet` = '{$wallet_amount}' WHERE `user_id` = {$wo['user']['user_id']} ");
                                $data['status'] = 200;
                                $data['url']    = Wo_SeoLink('index.php?link1=upgraded');
                            }
                        } else {
                            $data['message'] = $error_icon . $wo['lang']['something_wrong'];
                        }
                    } else {
                        $data['message'] = $error_icon . $wo['lang']['something_wrong'];
                    }
                } elseif ($_GET['type'] == 'fund') {
                    $amount             = $price;
                    //$notes              = "Doanted to " . mb_substr($fund->title, 0, 100, "UTF-8");
                    $notes              = mb_substr($fund->title, 0, 100, "UTF-8");
                    //$notes              = str_replace('{text}', mb_substr($fund->title, 0, 100, "UTF-8"), $wo['lang']['trans_doanted_to']);
                    $create_payment_log = mysqli_query($sqlConnect, "INSERT INTO " . T_PAYMENT_TRANSACTIONS . " (`userid`, `kind`, `amount`, `notes`) VALUES ({$wo['user']['user_id']}, 'DONATE', {$amount}, '{$notes}')");
                    $wallet_amount      = ($wo["user"]['wallet'] - $price);
                    $query_one          = mysqli_query($sqlConnect, "UPDATE " . T_USERS . " SET `wallet` = '{$wallet_amount}' WHERE `user_id` = {$wo['user']['user_id']} ");
                    $admin_com          = 0;
                    if (!empty($wo['config']['donate_percentage']) && is_numeric($wo['config']['donate_percentage']) && $wo['config']['donate_percentage'] > 0) {
                        $admin_com = ($wo['config']['donate_percentage'] * $amount) / 100;
                        $amount    = $amount - $admin_com;
                    }
                    $user_data = Wo_UserData($fund->user_id);
                    $db->where('user_id', $fund->user_id)->update(T_USERS, array(
                        'balance' => $user_data['balance'] + $amount
                    ));
                    $fund_raise_id           = $db->insert(T_FUNDING_RAISE, array(
                        'user_id' => $wo['user']['user_id'],
                        'funding_id' => $fund_id,
                        'amount' => $amount,
                        'time' => time()
                    ));
                    $post_data               = array(
                        'user_id' => Wo_Secure($wo['user']['user_id']),
                        'fund_raise_id' => $fund_raise_id,
                        'time' => time(),
                        'multi_image_post' => 0
                    );
                    $id                      = Wo_RegisterPost($post_data);
                    $notification_data_array = array(
                        'recipient_id' => $fund->user_id,
                        'type' => 'fund_donate',
                        'url' => 'index.php?link1=show_fund&id=' . $fund->hashed_id
                    );
                    Wo_RegisterNotification($notification_data_array);
                    $data = array(
                        'status' => 200,
                        'url' => $config['site_url'] . "/show_fund/" . $fund->hashed_id
                    );
                }
            }
        } else {
            $data['message'] = $error_icon . $wo['lang']['something_wrong'];
        }
        header("Content-type: application/json");
        echo json_encode($data);
        exit();
    }
```

## Second One

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
