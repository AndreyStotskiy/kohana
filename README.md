<?php foreach($groups as $group):

                foreach($group[1] as $part) :

                    $price_item = $part['price_items'][0];
echo $price_item->get_price_for_client() ;

Ця функция для сортировки, наприклад по-умолчанию передаються однi данi
з параметрами iншi

якщо визиваю так

$a = $price_item->get_price_for_client() ;
$b = $price_item->get_price_for_client(false,true,6) ;

то видае $a i $b однаково те що спочатку зверху в $a
(72)
(72)

якщо зминити мicцями то навпаки

$b = $price_item->get_price_for_client(false,true,6) ;
$a = $price_item->get_price_for_client() ;

(64)
(64)

задача зробити

(72)
(64)

endforeach;

endforeach;

?>

ось цi функции це формуеться цiна залежно вiд клиента

public function get_price() {
if(empty($this->_price)) $this->_price = ceil($this->price * $this->currency->ratio);
return $this->_price;
}

public function get_price_for_client($client = false, $standart = false, $discount_id = false) {
    if(empty($this->_price_for_client)) {
        $price = $this->get_price();

        $discount =  $this->get_discount_for_client($client, $standart, $discount_id);

        foreach($discount->discount_limits->find_all()->as_array() as $dl) {
            if($price > $dl->from && ($price <= $dl->to || $dl->to == 0)) {
                $price = round(($price * (100 + $dl->percentage) / 100), 0);

                break;
            }
        }
        $this->_price_for_client = $price;
    }

    return $this->_price_for_client;
}

public function get_discount_for_client($client = false, $standart = false, $discount_id = false) {
    if($discount_id) {
        $discount = ORM::factory('Discount', $discount_id);
    } elseif($client) {
        $discount = $client->discount;
    } else {
        if(!ORM::factory('Client')->logged_in() || $standart) {
            $discount = ORM::factory('Discount')->getStandart();
        } else {
            $discount = ORM::factory('Client')->get_client()->discount;
        }
    }

    return $discount;
}
