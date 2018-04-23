---
layout: post
title: Red Palito part 2
permalink: red-palito-part-2
published: true

---

Newly implemented:

- 180º turn mechanic
- Two types of enemies
- Enemies start to chase player on sight

![Screenshot]({{ site.baseurl }}/assets/red-palito-part-2-general.PNG)

So, v0.2 is finally out. I got stuck for a while trying to add models to the game. I managed to make some models (they had animations and everything), but the frame rate went way down. I decided to leave them as little boxes. And you know what? It looks quite nice in the end.

At some point during the development of v0.2, I decided to post the game on [r/IndieGaming](https://www.reddit.com/r/IndieGaming). It was my first post there, but people seem to like [it](https://www.reddit.com/r/IndieGaming/comments/8co4a9/first_game_and_first_post_on_this_subreddit/).

A lot of feedback came from Reddit, so I took notes and started implementing some of the ideas proposed. A few of them were really good and felt like a good step up from the previous version.

## 180º turn

This was the most requested feature from the feedback gathered. It was painfully slow to wait for the character to turn around the whole 180 degrees. You could even get killed waiting. Horrible.

This is how it looks after implementing the 180 turn.

![180 degree turn]({{ site.baseurl }}/assets/red-palito-part-2-180-turn.gif)

It is triggered by double pressing back. The technology wasn't there, so I created an object that keeps track of when the last `keyup` event was registered on the specific key.

``` js

window.addEventListener("keydown", e => {
	Input.keyState[e.keyCode || e.which] = true;
	if (Input.keyLastPressed[e.keyCode || e.which] != null) {
		let doublePressTimeout = $("#double-press-slider").val(); // miliseconds
		Input.keyDoublePress[e.keyCode || e.which] = new Date().getTime() - Input.keyLastPressed[e.keyCode || e.which].getTime() < doublePressTimeout;
	}
}, true);

window.addEventListener("keyup", e => {
	Input.keyState[e.keyCode || e.which] = false;
	Input.keyLastPressed[e.keyCode || e.which] = new Date();
}, true);

```

I also included a setting that allows the player to modify the interval between two presses.

## Two more types of enemies

Other suggestion I got from the post was to add more types of enemies. I was already working on two new enemies, so that one felt perfect.

The two new enemies were:

### Zoomies

![Zoomies]({{ site.baseurl }}/assets/red-palito-part-2-zoomie.PNG)

Zoomies are smaller and faster. When they see the player, they lock position and start charging for a moment. After that, they dash to the position they locked. If they hit the player, they die.

The dash is an interesting mechanic. I modified the `moveTowardPlayer` method of the new class, adding a dash option.

``` js

moveTowardPlayer() {
	// Normal movement. When the player is in range, start preparing dash
	if (!this.isDashing) {
		this.lookAtPosition();
		this.moveForward();
		if (settings.modelsEnabled) this.animationMixer.clipAction(this.animations.walk).play();

		if (this.position.distanceTo(player.position) <= this.dashDistance) {
			this.isDashing = true;
			this.distanceTraveled = 0;
			this.targetPosition = new THREE.Vector3(player.position.x, this.position.y, player.position.z);
		}
	} else {
		// Update the material used
		if (this.dashCountDown > 1) {
			this.Mesh.material = smallZombiePrepareMaterial;
		} else {
			this.Mesh.material = smallZombieDashMaterial;
		}

		// Charging dash
		if (this.dashCountDown > 0) {
			this.lookAtPosition(this.targetPosition);
			this.dashCountDown -= frameTime;
		} else {
			this.dashForward();
			if (this.distanceTraveled >= this.dashDistance) {
				this.isDashing = false;
				this.dashCountDown = this.dashTime;
			}
		}
	}
}

```

### Giants

![Giant]({{ site.baseurl }}/assets/red-palito-part-2-giant.PNG)

Giants are, needless to say, bigger and slower. They hit and tank much harder. Other than that, quite similar to the base enemy.


## Enemy vision

Enemies now are not always following the player. Each enemy has a perception distance. If the player is too far, they will wander around the map.

This solves a bit the problem of having the player *herding* the enemies into a neat group that just waits to be killed. Improvements can still —and will— be made.
