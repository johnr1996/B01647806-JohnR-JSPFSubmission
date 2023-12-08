# Element 5
For my fifth element, I have created a short game that involves the player controlling a character to push a ball around obstacles and into a goal. The element consists of three scenes:
1. Main Menu
2. Gameplay
3. Win Screen

# Menu Scene
When the element is first opened, the Main Menu scene is displayed. This displays the game's name, plus a short description of what the player has to do. Below this description is the button to begin the game. There is also an acoustic drum soundtrack that plays in the background, and will loop for as long as this scene is open.
```typescript
const menuMusic = new Sound("menuMusic", "./audio/DrumsLoop.wav", scene, null, {
          loop: true,
          autoplay: true,
          volume: 1
        });
```
Pressing the 'Press to start!' button will load the Gameplay scene.
```typescript
button.onPointerUpObservable.add(function() {
            console.log("THE BUTTON HAS BEEN CLICKED");
            menuMusic.stop();
            buttonClick.play();
            
            setSceneIndex(1);
        });
```

# Gameplay Scene
For my gameplay scene, I made a simple physics-based game where the player controls a character and has to push a ball around a small arena, avoiding obstacles and getting the ball into a goal.

## Audio
When the scene is loaded, the menu music is disabled and replaced with a desert ambience looping effect. This sound helped to portray the desert environment appropriately, while also not getting jarring or annoying with a repetitive song.

## User Interface
The UI is displayed on the bottom left of the screen, and displays the game's controls to remind the player.

## Camera
The player camera is locked in both its height and zoom levels, and can only be rotated and moved horizontally.
```typescript
    camera.attachControl(true);
    camera.upperBetaLimit = Math.PI / 4;  
    camera.lowerBetaLimit = Math.PI / 4;
    camera.upperRadiusLimit = 30;
    camera.lowerRadiusLimit = 30;
    return camera;
```

## Environment
The outer environment consists of rocky mountainous terrain, using a height map. This is used as the backdrop to the scene, and covers the disparity between the edge of the terrain and the skybox.
```typescript
function createTerrain(scene: Scene){
    const largeGroundMat = new StandardMaterial("largeGroundMat");
    largeGroundMat.diffuseTexture = new Texture("textures/00652_2x2_header1.png");
    const largeGround = MeshBuilder.CreateGroundFromHeightMap("largeGround",
    "https://assets.babylonjs.com/environments/villageheightmap.png",
    {width:150, height:150, subdivisions: 20, minHeight:0, maxHeight: 10});

    largeGround.material = largeGroundMat;
    return largeGround;}
  ```

The main area of the scene consists of a flat, sandy terrain, which is a collideable mesh where the gameplay takes place. This area is decorated with cacti plants around the edges, which are flat png sprites that are randomly generated within a set range of coordinates upon the scene's startup.
```typescript
function createTrees(scene: Scene){
    const spriteManagerTrees = new SpriteManager("treesManager", "textures/cacti.png", 2000, {width: 512, height: 1024}, scene);
    for (let i = 0; i < 100; i++) {
          const tree = new Sprite("tree", spriteManagerTrees);
          tree.position.x = Math.random() * (-60) + 30;
          tree.position.z = Math.random() * 18 + 21;
          tree.position.y = 1.5;
          tree.size = 2;
        }
    for (let i = 0; i < 100; i++) {
        const tree = new Sprite("tree", spriteManagerTrees);
        tree.position.x = Math.random() * (-60) + 30;
        tree.position.z = Math.random() * -19 - 19;
        tree.position.y = 1.5;
        tree.size = 2;
        }
    return spriteManagerTrees;
  }
```

## Arena
In the center of the sandy terrain is the arena, where the gameplay takes place. It is a rectangular area made up of walls, that contains obstacles and the goal, and is where the player and ball will spawn upon the scene's startup.

### Walls
The arena consists of two walls, both of them cloned and repositioned to create the 4 walls of the arena that surround the player. The walls all have the same yellow and black warning hazard texture applied to them, to signify that they are bounds of the arena.
```typescript
function createTopWall(scene: Scene, x: number, y: number, z: number){
    let box: Mesh = MeshBuilder.CreateBox("wallTop");
    const wallMaterial = new StandardMaterial("wall", scene);

        wallMaterial.backFaceCulling = false;
        wallMaterial.diffuseTexture = new Texture("textures/barrier.jpg", scene);
        wallMaterial.diffuseTexture.hasAlpha = true;
        wallMaterial.diffuseTexture.coordinatesMode = Texture.CUBIC_MODE;
        wallMaterial.diffuseColor = new Color3(1, 1, 1);
        box.material = wallMaterial;
  
        box.position.x = x;
        box.position.y = y;
        box.position.z = z;
        box.scaling._x = 36;
        box.scaling.y = 3;
        box.scaling._z = 0.2;
  
        let box2 = box.clone("box");
        box2.position.x = 0;
        box2.position.y = 2;
        box2.position.z = -18;
        box2.scaling._x = 36;
        box2.scaling.y = 3;
        box2.scaling._z = 0.2;
  
    const boxAggregate = new PhysicsAggregate(box, PhysicsShapeType.BOX, { mass: 0 }, scene);
    const box2Aggregate = new PhysicsAggregate(box2, PhysicsShapeType.BOX, { mass: 0 }, scene);

return box;
  }
```
### Obstacles
Inside the arena are four large immovable boxes, that act as the obstacles the player must push the ball around during gameplay. These are simple box meshes, with the same yellow and black hazard texture applied to them, to keep them aesthetically matching with the immovable walls.
```typescript
function createObstacle1(scene: Scene, x: number, y: number, z: number){
    let box: Mesh = MeshBuilder.CreateBox("obstacle");
  
    const obstacleMaterial = new StandardMaterial("wall", scene);
        obstacleMaterial.backFaceCulling = false;
        obstacleMaterial.diffuseTexture = new Texture("textures/barrier.jpg", scene);
        obstacleMaterial.diffuseTexture.hasAlpha = true;
        obstacleMaterial.diffuseTexture.coordinatesMode = Texture.CUBIC_MODE;
        obstacleMaterial.diffuseColor = new Color3(1, 1, 1);
        box.material = obstacleMaterial;
  
        box.position.x = x;
        box.position.y = y;
        box.position.z = z;
        box.scaling._x = 3;
        box.scaling.y = 3;
        box.scaling._z = 3;
  
    const boxAggregate = new PhysicsAggregate(box, PhysicsShapeType.BOX, { mass: 0 }, scene);

    return box;
  }

```

### Goal
The goal is a long rectangular box mesh with a transparent net texture applied to it, looking much like a metal wire fence. The goal has no mesh applied and both the player and ball can move straight through it.

## Game Objects
### Player Character
The player character is a blue dummy model that is controlled by the player using the WASD keyboard buttons. The character can walk, rotate, moonwalk and sprint.
```typescript
let keydown: boolean = false;
        if (keyDownMap["w"] || keyDownMap["ArrowUp"]) {
          mesh.moveWithCollisions(mesh.forward.scaleInPlace(speed));                
          keydown = true;
        }
        if (keyDownMap["a"] || keyDownMap["ArrowLeft"]) {
          mesh.rotate(Vector3.Up(), -rotationSpeed);
          keydown = true;
        }
        if (keyDownMap["s"] || keyDownMap["ArrowDown"]) {
          mesh.moveWithCollisions(mesh.forward.scaleInPlace(-speed));
          keydown = true;
        }
        if (keyDownMap["d"] || keyDownMap["ArrowRight"]) {
          mesh.rotate(Vector3.Up(), rotationSpeed);
          keydown = true;
        }
        if (keyDownMap["Shift"]) {
          speed = 0.04;
          keydown = true;
        }
        if (keyDownMap["Shift"] == false) {
          speed = 0.02;
        }
```
The character has a capsule collider on it's mesh, which is used to collide with the ground and environment objects. The character's mesh has a mass of 10 applied to it, which allows it to push the lighter ball around the arena. The character has both walking and idle animations that play and stop when appropriate.
```typescript
let isPlaying: boolean = false;
        if (keydown && !isPlaying) {
          if (!animating) {
            idleAnim = scene.stopAnimation(skeleton);
            walkAnim = scene.beginWeightedAnimation(skeleton, walkRange.from, walkRange.to, 1.0, true);
            animating = true;
          }
         if (animating) {
          isPlaying = true;
          }
        }
        else {
           if (animating && !keydown) {
            walkAnim = scene.stopAnimation(skeleton);
            idleAnim = scene.beginWeightedAnimation(skeleton, idleRange.from, idleRange.to, 1.0, true);
            animating = false;
            isPlaying = false;
           }
        }
```

### Ball
The ball is the main object of the game, as it controls the win condition for the player to complete the game. The ball is a simple white sphere object, with a sphere collider applied and a mass of 2, making it lighter than the player character and easily pushed around.

The ball also collides with everything in the scene except the goal. When the ball's mesh intersects with the goal's mesh, this triggers the game's win condition and the scene progresses to the win screen.
```typescript
scene.onBeforeRenderObservable.add(()=> { 
      if (ball.intersectsMesh(goal)){
        console.log("BALL IS IN");

        //put win code here
        inGoal = true;
        setSceneIndex(2);
      }

      if (inGoal == true)
      {
        console.log("in goal = true");
        ball.position._x = 0;
        ball.position._y = 2;
        ball.position._z = -8;
        return ball;
      }
    });
```

# Win Screen
When the player has succesfully pushed the ball into the goal, the gameplay scene transitions to this screen. This screen is just a camera shot of the skybox, with cenetered UI text that lets the player know they have completed the game, and tells them how to return to the main menu.
```typescript
function createDescriptionText(scene: Scene, name: String, x: string, y: string, advtex)
 {
  let tutText = new GUI.TextBlock();
  tutText.left = x;
  tutText.top = y;
  tutText.text = "You put the ball in the net and completed the game!
  \nRefresh the page to return to the main menu!";
  tutText.color = "white";
  tutText.fontSize = 20;
  advtex.addControl(tutText);
 }
```

