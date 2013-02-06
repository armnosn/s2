# Gamification

as known as

* "Point & Badges system"
* "Archievement & Reputation system"

Take an action > Get points > Get badges. Simple.

So, our logic was:

```php
if ( $this->Post->save($data) ) {
	$this->Session->setFlash(__('Your post has been saved.'));
    $this->redirect(array('controller' => 'posts', 'action' => 'index'));
}
```

When we about to implement points & badges system, the code might look like:

```php
if ( $this->Post->save($data) {

	if ( $this->checkForPoint() ) {
		$this->User->updatePoint('AddNewPost');
    }
    
    if ( $this->checkForBadges() ) {
    	$this->User->updateBadge();
    }
    // then redirect to another page
}
```

and `updatePoint()` function migth look something like:

```php
public function updatePoint($actionType = '') {
	// get points based on taked action
    $point = $this->__getPoints($actionType);
    
    // set data and save
    $this->User->set(array(
    	'id' => $userId,
        'points' => $point
    ));
    
    return $this->User->save();
}

// define all business rules for points (points received per action)
private function __getPoints($actionType) {
	switch ($actionType) {
    	case 'AddNewPost' :
        	$points = 10;
            break;
        case 'AddNewComment' :
        	$points = 1;
            break;
        ...    
    }
    return $points;
    
}
```

as well as `checkForBadge()` function, might have similar algorithm.

## Problems

- Business rules (which defines what actions get how many points or get what badges) are tied to application's logic (in the code). Checking for points and bagdes are parts of the app. So, let's say we change application flow, it could affect defined points/badges rules.
- Points & Badges rules cannot be taken out from the app easily since they are parts of application flow. Less reusability.
- It adds more complexity to the app.

## Ideas

- putting points & badges system into the app shouldn't affect application's flow.
- rules about getting points and badges should not live with application's main logic.
- points & badges could be turned on/off and the app still works.

## Approaches
1. CakePHP has come callback methods in Model and Controller classes. In this case, we can use Models' `afterSave()` callback function in the Model to do all points & badges stuffs.
  - This function is executed immediately after `Model::save()` function. Always.
  - So we can put functions like `checkPoints`, `checkBadges`, `updateUserPoints`, `UpdateUserBadges` here.

