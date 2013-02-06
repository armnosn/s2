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
- It adds more complexity to the app. This could cause endless function calls.

## Ideas

- putting points & badges system into the app shouldn't affect application's flow.
- rules about getting points and badges should not live with application's main logic.
- points & badges could be turned on/off and the app still works.
- and it is also easy to change rules for points and badges (like add/remove some rules or change the way it should work).

## Approaches

### 1. `afterSave()` callback method

CakePHP has come callback methods in Model and Controller classes. In this case, we can use Models' `afterSave()` callback function in the Model to do all points & badges stuffs.
  - This function is executed immediately after `Model::save()` function. Always.
  - So we can put functions like `checkPoints`, `checkBadges`, `updateUserPoints`, `UpdateUserBadges` here.

#### Problems with this approach
1. Points & Badges do not always happen when `Model::save()` is called. Regular tasks like viewing a post (there is `save()` called to update post's view count) or updating data from admin panel (in the future) should not execute points & badges logic.
2. Business rules are still defined partly in the app. In this case, some Models (or AppModel).

### 2. Use CakePHP's Event System

**The Observer Pattern**

Updating points and badges is a kind of `events` happens in the app. So we can create events and event listeners to integrate into our application. Example events are
- post new topic
- invite friends
- build profile

Each event does things differently. So we can create `event listener` (basically functions) to do things without affecting other events or main application flow.

I think this idea is good because
- It keeps application modular. Also separates business rules from application logic.
- When we want to change what the action `post new topic` does (like change points, or add more badges), it will be easy because we can go to modify the action's event listener function. Not change the whole logic.
- Application itself doesn't have to do much. Just trigger an event. The rest is event listener's work.

CakePHP has an interface [CakeEventListener](http://book.cakephp.org/2.0/en/core-libraries/events.html) 
