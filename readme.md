<p align="center">
  <a href="https://github.com/drcmda/learnwithjason"><img width="288" height="172" src="https://i.imgur.com/1atWRR3.gif" /></a>
  <a href="https://codesandbox.io/embed/react-three-fiber-suspense-gltf-loader-l900i"><img width="288" height="172" src="https://i.imgur.com/8xpKFkN.gif" /></a>
  <a href="https://codesandbox.io/embed/387z7o2zrq"><img width="288" height="172" src="https://i.imgur.com/YewcWL5.gif" /></a>
  <a href="https://codesandbox.io/embed/m7q0r29nn9"><img width="288" height="172" src="https://i.imgur.com/LDddjjC.gif" /></a>
  <a href="https://codesandbox.io/embed/jz9l97qn89"><img width="288" height="172" src="https://i.imgur.com/zrhe5Jc.gif" /></a>
  <a href="https://codesandbox.io/embed/kky7yk087v"><img width="288" height="172" src="https://i.imgur.com/jemlXzE.gif" /></a>
  <a href="https://codesandbox.io/embed/ly0oxkp899"><img width="288" height="172" src="https://i.imgur.com/vjmDwpS.gif" /></a>
  <a href="https://codesandbox.io/embed/9y8vkjykyy"><img width="288" height="172" src="https://i.imgur.com/tQi753C.gif" /></a>
  <a href="https://codesandbox.io/embed/y3j31r13zz"><img width="288" height="172" src="https://i.imgur.com/iFtjKHM.gif" /></a>
</p>
<p align="middle">
  <i>These demos are real, you can click them! They contain the full code, too.</i>
</p>

    npm install three react-three-fiber

React-three-fiber is a small React renderer for Threejs on the web and react-native. Why, you might ask? React was made to drive complex tree structures, it makes just as much sense for Threejs as for the DOM. Building a dynamic scene graph becomes so much easier because you can break it up into declarative, re-usable components with clean, reactive semantics. This also opens up the ecosystem, you can now apply generic packages for state, animation, gestures and so on.

# What it looks like ...

Copy the following into a project to get going. [Here's the same](https://codesandbox.io/s/rrppl0y8l4) running in a code sandbox.

```jsx
import { Canvas, useFrame } from 'react-three-fiber'

function Thing() {
  const ref = useRef()
  useFrame(() => (ref.current.rotation.z += 0.01))
  return (
    <mesh
      ref={ref}
      onClick={e => console.log('click')}
      onPointerOver={e => console.log('hover')}
      onPointerOut={e => console.log('unhover')}>
      <planeBufferGeometry attach="geometry" args={[1, 1]} />
      <meshBasicMaterial attach="material" color="hotpink" opacity={0.5} transparent />
    </mesh>
  )
}

<Canvas>
  <Thing />
</Canvas>
```

# Canvas

The `Canvas` object is your portal into Threejs. It renders Threejs elements, *not DOM elements*!

```jsx
<Canvas
  children                      // Either a function child (which receives state) or regular children
  gl                            // Props that go into the default webGL-renderer
  camera                        // Props that go into the default camera
  raycaster                     // Props that go into the default raycaster
  shadowMap                     // Props that go into gl.shadowMap, can also be set true for PCFsoft
  vr = false                    // Switches renderer to VR mode, then uses gl.setAnimationLoop
  orthographic = false          // Creates an orthographic camera if true
  pixelRatio = undefined        // You could provide window.devicePixelRatio if you like 
  invalidateFrameloop = false   // When true it only renders on changes, when false it's a game loop
  updateDefaultCamera = true    // Adjusts default camera on size changes
  onCreated                     // Callback when vdom is ready (you can block first render via promise)
  onPointerMissed />            // Response for pointer clicks that have missed a target
```

You can give it additional properties like style and className, which will be added to the container (a div) that holds the dom-canvas element.

# Defaults that the canvas component sets up

Canvas will create a *translucent WebGL-renderer* with the following properties: `antialias, alpha, setClearAlpha(0)`

A default *perspective camera*: `fov: 75, near: 0.1, far: 1000, position.z: 5`

A default *orthographic camera* if Canvas.orthographic is true: `near: 0.1, far: 1000, position.z: 5`

A default *shadowMap* if Canvas.shadowMap is true: `type: PCFSoftShadowMap`

A default *scene* (into which all the JSX is rendered) and a *raycaster*.

You do not have to use any of these objects, look under "receipes" down below if you want to bring your own.

# Objects and properties

You can use [Threejs's entire object catalogue and all properties](https://threejs.org/docs). When in doubt, always consult the docs.

You could lay out an object like this:

```jsx
<mesh
  visible
  userData={{ test: "hello" }}
  position={new THREE.Vector3(1, 2, 3)}
  rotation={new THREE.Euler(0, 0, 0)}
  geometry={new THREE.SphereGeometry(1, 16, 16)}
  material={new THREE.MeshBasicMaterial({ color: new THREE.Color('hotpink'), transparent: true })} />
```

The problem is that all of these properties will be re-created on every render pass. Instead, you should define properties declaratively.

```jsx
<mesh visible userData={{ test: "hello" }} position={[1, 2, 3]} rotation={[0, 0, 0]}>
  <sphereGeometry attach="geometry" args={[1, 16, 16]} />
  <meshStandardMaterial attach="material" color="hotpink" transparent />
</mesh>
```

#### Shortcuts (set)

All properties that have a `.set()` method can be given a shortcut. For example [THREE.Color.set](https://threejs.org/docs/index.html#api/en/math/Color.set) can take a color string, hence instead of `color={new THREE.Color('hotpink')}` you can do `color="hotpink"`. Some `set` methods take multiple arguments ([THREE.Vector3.set](https://threejs.org/docs/index.html#api/en/math/Vector3.set)), so you can pass an array `position={[100, 0, 0]}`.

#### Shortcuts and non-Object3D stow-away

Stow away non-Object3D primitives (geometries, materials, etc) into the render tree so that they become managed and reactive. They take the same properties they normally would, constructor arguments are passed with `args`. Using the `attach` property objects bind automatically to their parent and are taken off it once they unmount.

You can nest primitive objects, too, which is good for awaiting async textures and such. You could use React-suspense if you wanted!

```jsx
<meshBasicMaterial attach="material">
  <texture attach="map" image={img} onUpdate={self => img && (self.needsUpdate = true)} />
```

Sometimes attaching isn't enough. For example, this code attaches effects to an array called "passes" of the parent `effectComposer`. Note the use of `attachArray` which adds the object to the target array and takes it out on unmount:

```jsx
<effectComposer>
  <renderPass attachArray="passes" scene={scene} camera={camera} />
  <glitchPass attachArray="passes" renderToScreen />
```

You can also attach to named parent properties using `attachObject={[target, name]}`, which adds the object and takes it out on unmount. The following adds a buffer-attribute to parent.attributes.position. 

```jsx
<bufferGeometry attach="geometry">
  <bufferAttribute attachObject={['attributes', 'position']} count={v.length / 3} array={v} itemSize={3} />
```

#### Piercing into nested properties

If you want to reach into nested attributes (for instance: `mesh.rotation.x`), just use dash-case:

```jsx
<mesh rotation-x={1} material-uniforms-resolution-value={[1 / size.width, 1 / size.height]} />
```

#### Putting already existing objects into the scene-graph

You can use the `primitive` placeholder for that. You can still give it properties or attach nodes to it.

```jsx
const mesh = new THREE.Mesh()
return <primitive object={mesh} position={[0, 0, 0]} />
```

#### Using 3rd-party (non THREE namespaced) objects in the scene-graph

The `extend` function extends three-fibers catalogue of known native JSX elements.

```jsx
import { extend } from 'react-three-fiber'
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer'
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass'
extend({ EffectComposer, RenderPass })

<effectComposer>
  <renderPass />
```

# Events

Threejs objects that implement their own `raycast` method (meshes, lines, etc) can be interacted with by declaring events on the object. We support pointer events ([you need to polyfill them yourself](https://github.com/jquery/PEP)), clicks and wheel-scroll. Events contain the browser event as well as the Threejs event data (object, point, distance, etc).

Additionally there's a special `onUpdate` that is called every time the object gets fresh props, which is good for things like `self => (self.verticesNeedUpdate = true)`.

```jsx
<mesh
  onClick={e => console.log('click')}
  onWheel={e => console.log('wheel spins')}
  onPointerUp={e => console.log('up')}
  onPointerDown={e => console.log('down')}
  onPointerOver={e => console.log('hover')}
  onPointerOut={e => console.log('unhover')}
  onPointerMove={e => console.log('move')}
  onUpdate={self => console.log('props have been updated')} />
```

#### Propagation and capturing

```jsx
  onPointerDown={e => {
    // Only the mesh closest to the camera will be processed
    e.stopPropagation()
    // You may optionally capture the target
    e.target.setPointerCapture(e.pointerId)
  }}
  onPointerUp={e => {
    e.stopPropagation()
    // Optionally release capture
    e.target.releasePointerCapture(e.pointerId)
  }}
```

# Hooks

Hooks can only be used **inside** the Canvas element because they rely on context! You cannot expect something like this to work:

```jsx
funciton App() {
  const { size } = useThree() // This will just crash
  return (
    <Canvas>
      <mesh>
```

Do this instead:

```jsx
funciton SomeComponent() {
  const { size } = useThree()
  return <mesh />
}
        
funciton App() {
  return (
    <Canvas>
      <SomeComponent />
```

#### useThree()

This hooks gives you access to all the basic objects that are kept internally, like the default renderer, scene, camera. It also gives you the current size of the canvas in screen and viewport coordinates.

```jsx
import { useThree } from 'react-three-fiber'

const { 
  gl,               // WebGL renderer
  canvas,           // canvas the dom element that was created
  scene,            // Default scene
  camera,           // Default camera
  size,             // Bounds of the view (which stretches 100% and auto-adjusts)
  viewport,         // Bounds of the viewport in 3d units + factor (size/viewport)
  aspect,           // Aspect ratio (size.width / size.height)
  invalidate,       // Invalidates a single frame (for <Canvas invalidateFrameloop />)
  intersect,        // Calls onMouseMove handlers for objects underneath the cursor
  setDefaultCamera  // Sets the default camera
} = useThree()
```

#### useFrame(callback, priority=0)

When you're running effects, postprocessings, controls, etc that need to get updated every frame. You receive the internal state as well, which is the same as what you would get from useThree.

```jsx
import { useFrame } from 'react-three-fiber'

// Subscribes to the render-loop, gets cleaned up automatically when the component unmounts
useFrame(state => console.log("I'm in the render-loop"))

// Add a priority as the 2nd argument and you have to take care of rendering yourself
// If you have multiple frames that render, they are ordered after the priority you give it
useFrame(({ gl, scene, camera }) => gl.render(scene, camera), 1)
```

#### useResource(optionalRef=undefined)

When you want to share and re-use resources. `useResource` creates a ref and re-renders the component when it becomes available next frame.

```jsx
import { useResource } from 'react-three-fiber'

const [ref, material] = useResource()
return (
  <meshBasicMaterial ref={ref} />
  {material && (
    <mesh material={material} />
    <mesh material={material} />
    <mesh material={material} />
```

#### useUpdate(callback, dependencies, optionalRef=undefined)

When objects need to be updated imperatively.

```jsx
import { useUpdate } from 'react-three-fiber'

const ref = useUpdate( 
  geometry => {
    geometry.addAttribute('position', getVertices(x, y, z))
    geometry.attributes.position.needsUpdate = true
  }, 
  [x, y, z], // execute only if these properties change
)
return <bufferGeometry ref={ref} />
```

#### useLoader(loader, url, [extensions]) (experimental!)

When you want to write out a loaded object declaratively, where you get to lay events on objects, alter materials, etc. It loads a file (which must be present in your /public folder) and caches it. It returns an array of geometry/material pairs.

```jsx
import { useLoader } from 'react-three-fiber'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader'

const model = useLoader(GLTFLoader, '/spaceship.gltf', loader => {
  const dracoLoader = new DRACOLoader()
  dracoLoader.setDecoderPath('/draco-gltf/')
  loader.setDRACOLoader(dracoLoader)
}))
return model.map(({ geometry, material }) => (
  <mesh key={geometry.uuid} geometry={geometry} castShadow>
    <meshStandardMaterial attach="material" map={material.map} roughness={1} />
```

# Additional exports

```jsx
import {
  addEffect,              // Adds a global callback which is called each frame
  invalidate,             // Forces view global invalidation
  apply,                  // Extends the native-object catalogue
  createPortal,           // Creates a portal (it's a React feature for re-parenting)
  render,                 // Internal: Renders three jsx into a scene
  unmountComponentAtNode, // Internal: Unmounts root scene
  applyProps,             // Internal: Sets element properties
} from 'react-three-fiber'
```

# Further information

Recipes and FAQ: [/react-three-fiber/recipes.md](https://github.com/react-three-fiber/react-three-fiber/blob/master/recipes.md)

Tools (GLTF-to-JSX etc): [/react-three-fiber/tools](https://github.com/react-spring/react-three-fiber/tools)

React-podcast episode: https://reactpodcast.simplecast.fm/56

Learn-with-jason: https://www.youtube.com/watch?v=1rP3nNY2hTo

# Contributions

If you like this project, please consider helping out. All contributions are welcome as well as donations to [Opencollective](https://opencollective.com/react-three-fiber), or in crypto:

BTC: 36fuguTPxGCNnYZSRdgdh6Ea94brCAjMbH

ETH: 0x6E3f79Ea1d0dcedeb33D3fC6c34d2B1f156F2682

#### Sponsors

<a href="https://opencollective.com/react-three-fiber/sponsor/0/website" target="_blank">
  <img src="https://opencollective.com/react-three-fiber/sponsor/0/avatar.svg"/>
</a>
<a href="https://opencollective.com/react-three-fiber/sponsor/1/website" target="_blank">
  <img src="https://opencollective.com/react-three-fiber/sponsor/1/avatar.svg"/>
</a>

#### Backers

Thank you to all our backers! 🙏

<a href="https://opencollective.com/react-three-fiber#backers" target="_blank">
  <img src="https://opencollective.com/react-three-fiber/backers.svg?width=890"/>
</a>

##### Contributors

This project exists thanks to all the people who contribute.

<a href="https://github.com/react-spring/react-three-fiber/graphs/contributors">
  <img src="https://opencollective.com/react-three-fiber/contributors.svg?width=890" />
</a>
