---
layout: post
title: Red Palito part 1
permalink: red-palito-part-1
published: true
---

[Part 1]({{ site.baseurl}}/red-palito-part-1)
[Part 2]({{ site.baseurl}}/red-palito-part-2)

Current state of the game:

- Basic logic implemented
- Enemies chase player
- A few weapons to chose from
- Weapon and HP drops
- Basic animations

![Screenshot]({{ site.baseurl }}/assets/red-palito-part-1-general.PNG)

It has been a couple of months since I started this project ([first commit](https://github.com/JuanFerrer/red-palito/commit/8e5d3ae9d85b8d5306ec87a62c251ddb7351c585) on 24 June) and it starts to look like a game. Many challenges have come up, and here I explain how they were solved.

## Bullet orientation

Correctly orienting the bullets was one of the first problems I encountered. Projectiles were being shot from the player towards the target. To get that nice stretched out bullet that we love to see in games, I had to give some length to the bullet. The problem now was to make it *face* the target.

![Long shot capture]({{ site.baseurl }}/assets/red-palito-part-1-long-shot.PNG)

So, on spawn, I position the bullet, reset its lifetime and make it look towards the target.

``` js
spawn(pos, dir, acc, sp = 1.8, lt = this.initialLifeTime) {
	this.direction = dir;
	this.isAlive = true;
	this.lifeTime = lt;
	this.speed = sp;
	this.Mesh.position.set(pos.x, 1, pos.z);
	this.orient(acc);
}
```

``` js
orient(acc) {
	const randX = Math.max((Math.random() - acc) / 20, 0);
	const randZ = Math.max((Math.random() - acc) / 20, 0);

	this.direction.add(new THREE.Vector3(randX, 0, randZ));
	this.Mesh.quaternion.setFromUnitVectors(new THREE.Vector3(0, 1, 0), this.direction.normalize());
}
```

## Get weapons from file

When the player object is created, some of the variables require information about the weapons he can use. So, the weapons need to be loaded before that.

I ended up going for asynchronous loading of the weapons. Get a handler to the parse object and, when ready, load everything else.

``` js
let parseResult = parseJSONToVar("weapons.json", "weapons", weapons);

parseResult.then(function () {
	Audio.loadWeaponSounds();
	Audio.loadPickupSounds();
	Audio.loadEnemySounds();
	Audio.loadPlayerSounds();
	listener.setMasterVolume(settings.masterVolume);
	setupPlayer();
	setGunFlare();

	player.Mesh.add(listener);
	});
```

``` js
function parseJSONToVar(file, object, copyTo) {
	return $.getJSON(file, function (data) {
		data[object].forEach(v => {
			copyTo.push(v);
		});
	});
}
```

## Using single light

Lastly, I found that the FPS were still low (they can be improved further, but I'll wait a bit for that). It was usually oscillating the 60, but sometimes dropping to 20. I tried [different things](http://learningthreejs.com/blog/2011/09/16/performance-caching-material/) to fix it, but in the end, lowering the amount of lights in the scene was key.

---

I will post again when I get to v0.2.






