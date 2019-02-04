# Тестовое задание по Yii2

## Дано:

- Любой (пустой) проект на Yii2

- Табличка в PostgreSQL описанная такой миграцией:

```php
$this->createTable('{{%account}}', [
    'id'                    => $this->pk(),
    'email'                 => $this->string(),
    'password'              => $this->string(),
    'settings'              => $this->json('{"color":"#00FF00","phone":null}'),
    'options'               => $this->json('{"height":100,"width":100}'),
]);
```

- Модель Account:
```php
/**
 * Class Account
 *
 * @package app\models
 * @property int id
 * @property string email
 * @property string password
 * @property SettingsJson settings
 */
 
class Account extends TrickyModel {

    public function rules() {
      return [
        ['id', 'number','integerOnly'=>true],
        ['email', 'email'],
        ['password', 'string'],
        ['settings', JsonValidator::class, 'model' => SettingsJson:class],
      ];
    }
    
}
```

- Класс SettingsJson:
```php
class SettingsJson extends \yii\base\Model implements \JsonSerializable {

  public $color = "#00FF00";
  
  public $phone = null;
  
  public function rules() {
      return [
        ['phone', PhoneValidator::class],
        ['color', ColorValidator::class],
      ];
  }

  public function jsonSerialize()
  {
     return $this->toArray();
  }
}
```

## Задача:
       
Реализовать классы 
- **TrickyModel**, 
- **JsonValidator**, 
- PhoneValidator
- **ColorValidator** 
с которыми возможен данный сценарий:

```php
$account = new Account();
$account->id = 1;
$account->email               = 'test@test.com';
$account->password            = '12345';
$account->options             = ['height' => 100, 'width' => 100];   

$account->options['height']   = 200;           // 1. не должно вызывать ошибку
$account->settings->color     = '#00AAAA';     // 1. не должно вызывать ошибку
$account->settings->phone     = '9009333333';  // 1. не должно вызывать ошибку
$account->save(); 


$account2 = Account::findOne(1);

print_r($account->settings->color);     // 2. Должно вывести '#00AAAA'
print_r($account->options);             // 2. Должно вывести массив ['hegiht' => 200, 'width' => 100]

$account2->settings->color = 'test123';
$account2->validate(); 

print_r($account2->errors);              // 3. Выведет массив с ошибками где будет: "Ошибка валидации settings параметра color - 'test123' не является цветом"

$account2->settings->color   = '#AAAAAA';
$account2->options['height'] = 300;  
$account2->save();
```

## Критерий успешности решения тестового задания

1) Не было ошибок присвоения значений
2) Были выведены измененные значеня settings и options
3) Была выведен ошибка "Ошибка валидации settings параметра color - 'test123' не является цветом"
4) В результате приведенного выше кода в базе должна остаться запись с такими значениями:

```yml
id:         1, 
email:      'test@test.com', 
password:   '12345', 
settings:   '{"color":"#AAAAAA","phone":"8000000000"}', 
options:    '{"height":300,"width":100}'
```

**Важно! Запрещено использовать временную переменную вида:**
```php
$options = $account->options;
$options['height'] = 200;
$account->options = $options
```
        
