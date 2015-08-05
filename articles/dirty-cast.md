Hello. I notice enough strange behavior. 
I have 2 classes:
```
/**
* @property array $gateway
*/
class User extends Illuminate\Database\Eloquent\Model {

    protected $casts = [
        'gateway' => 'array',
    ];
    
    public static function boot()
    {
        parent::boot();
        static::saving(function (User $user) {
            $userHistory = UserHistory::create(
        array_merge([
            'comment'    => $user>_comment ?: '',
            'client_id'  => $user->user_id,
            'date_added' => date('Y-m-d H:i:s'),
            'operator'   => $user->_operator_id,
        ], $user->getDirty()
        )
        );
        
        return true;
    });
}

/**
* @property array $gateway
*/
class UserHistory extends  Illuminate\Database\Eloquent\Model {

    protected $casts = [
        'gateway' => 'array',
    ];

}

```

At class  UserHistory I save all changes of User objects.  But at class UserHistory at field gateway I get not json but string into json. That appears because of `getDirty()` return directly data from attributes array.

I believe that this is unexpected action or even bug. 


```

        static::saving(function (Client $client) {
            if ($client->isNeedChangeHistory) {
                /**
                 * @var Client $client
                 */
                $dirtyAttributes = $client->getDirty();
                if (isset($dirtyAttributes['gateway'])) {
                    $dirtyAttributes['gateway'] = json_decode($dirtyAttributes['gateway']);
                }
                $clientHistory = ClientHistory::create(
                    array_merge([
                        'comment'    => $client->_comment ?: '',
                        'client_id'  => $client->client_id,
                        'date_added' => date('Y-m-d H:i:s'),
                        'operator'   => $client->_operator_id
                    ], $dirtyAttributes)
                );
                if (!$clientHistory->exists) {
                    throw new Exception('Can`t save balance sender', TransferValidationErrors::UNKNOWN_ERROR);
                }
            }
            return true;
        });

```