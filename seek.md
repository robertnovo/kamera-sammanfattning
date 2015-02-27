# Seek
![](/img/pos_velocity_vectors.png)

Figuren simulerar kameran vid position (x, y, z) med en hastighet (a, b, c) (Euler integration)

```javascript
position = position.add(velocity); // Vec3
```

Riktningen på hastighets-vektorn kontrollerar vart kameran är på väg, medan magnituden (längden) på vektorn bestämmer hur mycket den rör sig per frame. Ju längre, desto snabbare rör sig kameran.

Hastighets-vektorn kan förkortas (truncated) för att försäkra sig om att den inte är större än ett visst värde, vanligtvis max-velocity.
```javascript
function truncate(vector, maxVelocity) {
	var i = maxVelocity / vector.length();
	i = (i < 1.0) ? i : 1.0;
	vector.multiplyScalar(i);
}
```

```javascript
// velocity-vektorn
velocity = target.sub(position).normalize().multiplyScalar(maxVelocity);
```

Utan steering-force beskrivs bara raka vägar som direkt byter riktning när kameran rör sig, vilket ger abrupta övergångar mellan den nuvarnade rutten och den nya.


### Beräkna krafter
Tanken är att introducera styr-krafter som påverkar kamerans rörelse. Utan dessa hade kameran följt en rak linje. Beroende på styr-krafterna så flyttas kameran i olika riktningar.

#### Seek-beteendet
Tack vare steering-krafterna som appliceras vid varje frame, så justeras hastigheten mjukt, då den försöker nå sin destination.

![](/img/steering_forces.png)

Seek-beteendet inkluderar två krafter, *desired velocity* och *steering*

```javascript
desiredVelocity = target.sub(position).multiplyScalar(maxVelocity);
steering = desiredVelocity.sub(velocity);
```

### Lägga på krafter
Efter att steering-kraften har beräknas, så måste den läggas till på kamerans hastighetsvektor (velocity). Detta resulterar i en "seek-path" enligt figuren.

![](/img/steering_forces_seek_path.png)

Ser ut så här:
```javascript
steering = truncate(steering, maxForce);
steering = steering.divideScalar(cameraMass);

velocity = trunacte(velocity.add(steering), maxSpeed);
position = position.add(velocity);
```

* Truncate gör att det inte finns krafter på kameran som den inte kan hantera.
* Steering-kraften delas med kamerans massa för att producera olika hastigheter för olika föremål.
