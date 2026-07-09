# i2i-Academy-Three.js-React-Three-Fiber-Game-Development-1

This project is a 3D endless runner game built with React, Vite, and React Three Fiber. The main goal is to demonstrate core game mechanics in the browser, including smooth player movement, dynamic obstacle generation, and real time collision detection. It also features a custom 2D UI to track the player's score and manage the game over state.

Step 1 - Development Environment & Setup:
  I used Vite with the React template. I opened the terminal in my project folder and ran the following command to create the base structure;

     npm create vite@latest . -- --template react

  After downloaded the Vita, I needed to install the required 3D libraries for R3F as stated in the instructions. I ran this command to download the packages:

    npm install three @react-three/fiber @react-three/drei


Step 2 - Core Game Mechanics & Implementation:

Scene Infrastructure: 
  As requested in the instructions, I initialized a basic 3D scene and after running the code we see just a red cube with perspective camera, WebGL Renderer, flat playing field and balanced lighting.

    import { Canvas } from '@react-three/fiber'
    import './App.css'
    
    // This component represents the main character controlled by the player
    function Player() {
        return (
            // position sets the starting point, and castShadow allows the object to drop shadows
            <mesh position={[0, 0.5, 0]} castShadow>
                {/* boxGeometry defines the 3D shape with specific width, height, and depth */}
                <boxGeometry args={[1, 1, 1]} />
                {/* meshStandardMaterial reacts to lights and applies a crimson red color to the cube */}
                <meshStandardMaterial color="crimson" />
            </mesh>
        )
    }
    
    // This component creates the main floor or track for the game
    function Ground() {
        return (
            // receiveShadow ensures that the shadows of other objects are visible on this surface
            <mesh position={[0, -0.05, 0]} receiveShadow>
                {/* boxGeometry is adjusted to create a long, flat running platform (10 units wide, 40 units long) */}
                <boxGeometry args={[10, 0.1, 40]} />
                {/* A simple dark gray color is selected to keep the design clean */}
                <meshStandardMaterial color="#444444" />
            </mesh>
        )
    }
    
    // The core component that initializes and manages the whole 3D world
    function App() {
        return (
            // Canvas creates the WebGL scene. camera sets up the perspective and view angles
            <Canvas shadows camera={{ position: [0, 5, 10], fov: 60 }}>
    
                {/* ambientLight gives a soft, general brightness to the scene so it is not pitch black */}
                <ambientLight intensity={0.4} />
    
                {/* directionalLight acts like the sun, shining from a distance to create realistic light and shadow details */}
                <directionalLight
                    position={[5, 10, 5]}
                    intensity={1.2}
                    castShadow
                    // shadow-mapSize configures the shadow resolution for a good balance of performance and visuals
                    shadow-mapSize={[1024, 1024]}
                />
    
                {/* Adding the player and the ground components into the 3D Canvas */}
                <Player />
                <Ground />
    
            </Canvas>
        )
    }
    
    export default App


Player Controls - Game Loop & Obstacles:
  To control the player I added event listeners that detect keyboard inputs from the Arrow keys or A/D keys. I used the useFrame hook to update the red cube's position smoothly on the X-axis in every frame. I also added coordinate limits to ensure the player stays on track without falling the edges.

    import { useRef, useEffect } from 'react'
    import { Canvas, useFrame } from '@react-three/fiber'
    import './App.css'
    
    // This component represents the main character controlled by the player
    function Player() {
        // We use useRef to directly access and modify the 3D mesh without reloading the whole page
        const meshRef = useRef()
        // This reference keeps track of which keys are currently being pressed
        const keys = useRef({ left: false, right: false })
    
        // useEffect runs once when the game starts to set up keyboard event listeners
        useEffect(() => {
            // Function to check if movement keys are pressed down
            const handleKeyDown = (e) => {
                if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.current.left = true
                if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.current.right = true
            }
    
            // Function to check if movement keys are released
            const handleKeyUp = (e) => {
                if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.current.left = false
                if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.current.right = false
            }
    
            // Adding the listeners to the browser window
            window.addEventListener('keydown', handleKeyDown)
            window.addEventListener('keyup', handleKeyUp)
    
            // Cleaning up the listeners when the component is removed to prevent errors
            return () => {
                window.removeEventListener('keydown', handleKeyDown)
                window.removeEventListener('keyup', handleKeyUp)
            }
        }, [])
    
        // useFrame runs on every single frame (like a Game Loop) to update the player's position
        useFrame((state, delta) => {
            // delta makes sure the movement speed is the same on all computers
            const speed = 5
    
            // Move left or right based on the pressed keys
            if (keys.current.left) {
                meshRef.current.position.x -= speed * delta
            }
            if (keys.current.right) {
                meshRef.current.position.x += speed * delta
            }
    
            // Keep the player inside the ground limits so they don't fall off the edge
            if (meshRef.current.position.x < -4.5) meshRef.current.position.x = -4.5
            if (meshRef.current.position.x > 4.5) meshRef.current.position.x = 4.5
        })
    
        return (
            // We attach the meshRef here to control this specific cube
            <mesh ref={meshRef} position={[0, 0.5, 0]} castShadow>
                <boxGeometry args={[1, 1, 1]} />
                <meshStandardMaterial color="crimson" />
            </mesh>
        )
    }
    
    // This component creates the main floor or track for the game
    function Ground() {
        return (
            <mesh position={[0, -0.05, 0]} receiveShadow>
                <boxGeometry args={[10, 0.1, 40]} />
                <meshStandardMaterial color="#444444" />
            </mesh>
        )
    }
    
    // The core component that initializes and manages the whole 3D world
    function App() {
        return (
            <Canvas shadows camera={{ position: [0, 5, 10], fov: 60 }}>
                <ambientLight intensity={0.4} />
                <directionalLight
                    position={[5, 10, 5]}
                    intensity={1.2}
                    castShadow
                    shadow-mapSize={[1024, 1024]}
                />
    
                <Player />
                <Ground />
            </Canvas>
        )
    }
    
    export default App

Creating Obstacles:

    // Importing necessary hooks from React and React Three Fiber
    import { useRef, useEffect } from 'react'
    import { Canvas, useFrame } from '@react-three/fiber'
    import './App.css'

    // This component represents the main character controlled by the player
    function Player() {
        const meshRef = useRef()
        const keys = useRef({ left: false, right: false })

        useEffect(() => {
            const handleKeyDown = (e) => {
                if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.current.left = true
                if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.current.right = true
            }

            const handleKeyUp = (e) => {
                if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.current.left = false
                if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.current.right = false
            }

            window.addEventListener('keydown', handleKeyDown)
            window.addEventListener('keyup', handleKeyUp)

            return () => {
                window.removeEventListener('keydown', handleKeyDown)
                window.removeEventListener('keyup', handleKeyUp)
            }
        }, [])

        useFrame((state, delta) => {
            const speed = 7

            if (keys.current.left) meshRef.current.position.x -= speed * delta
            if (keys.current.right) meshRef.current.position.x += speed * delta

            if (meshRef.current.position.x < -4.5) meshRef.current.position.x = -4.5
            if (meshRef.current.position.x > 4.5) meshRef.current.position.x = 4.5
        })

        return (
            <mesh ref={meshRef} position={[0, 0.5, 0]} castShadow>
                <boxGeometry args={[1, 1, 1]} />
                <meshStandardMaterial color="crimson" />
            </mesh>
        )
    }

    // This component creates moving obstacles that the player must avoid
    function Obstacle({ startZ, startX }) {
        const meshRef = useRef()

        // useFrame updates the obstacle position continuously to create movement
        useFrame((state, delta) => {
            // The obstacle moves towards the camera (positive Z axis)
            const speed = 15
            meshRef.current.position.z += speed * delta

            // If the obstacle passes the camera (z > 5), reset it to the far end of the track
            if (meshRef.current.position.z > 5) {
                meshRef.current.position.z = -40
                // Randomize the X position (left or right) to make the game unpredictable
                meshRef.current.position.x = (Math.random() - 0.5) * 8
            }
        })

        return (
            // position uses the initial startX and startZ props provided in the App component
            <mesh ref={meshRef} position={[startX, 0.5, startZ]} castShadow>
                <boxGeometry args={[1, 1, 1]} />
                {/* We use a blue color to easily distinguish enemies from the player */}
                <meshStandardMaterial color="dodgerblue" />
            </mesh>
        )
    }

    // This component creates the main floor or track for the game
    function Ground() {
        return (
            <mesh position={[0, -0.05, 0]} receiveShadow>
                <boxGeometry args={[10, 0.1, 40]} />
                <meshStandardMaterial color="#444444" />
            </mesh>
        )
    }

    // The core component that initializes and manages the whole 3D world
    function App() {
        return (
            <Canvas shadows camera={{ position: [0, 5, 10], fov: 60 }}>
                <ambientLight intensity={0.4} />
                <directionalLight
                    position={[5, 10, 5]}
                    intensity={1.2}
                    castShadow
                    shadow-mapSize={[1024, 1024]}
                />

                <Player />
                <Ground />

                {/* Spawning multiple obstacles at different starting distances and lanes */}
                <Obstacle startX={0} startZ={-10} />
                <Obstacle startX={-3} startZ={-20} />
                <Obstacle startX={3} startZ={-30} />
                <Obstacle startX={-1} startZ={-40} />

            </Canvas>
        )
    }

    export default App


Collision Detection - State Management & UI Overlay:
  I combined the collision and UI systems into a single functional structure. For collision detection the system continuously calculates the mathematical distance between the player and obstacles on the X and Z axes, triggering a Game Over if they touch. To manage this logic, I used a global state that tracks the live score and the game's current status. Finally I added HTML overlay on top of the 3D canvas to display the real-time score and show a restart screen when the game ends. 

    import { useRef, useEffect, useState } from 'react'
    import { Canvas, useFrame } from '@react-three/fiber'
    import './App.css'
    
    // Simple global state to handle game logic and share data between components
    const gameState = {
        playerX: 0,
        gameOver: false,
        score: 0
    }
    
    function Player() {
        const meshRef = useRef()
        const keys = useRef({ left: false, right: false })
    
        useEffect(() => {
            const handleKeyDown = (e) => {
                if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.current.left = true
                if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.current.right = true
            }
            const handleKeyUp = (e) => {
                if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.current.left = false
                if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.current.right = false
            }
    
            window.addEventListener('keydown', handleKeyDown)
            window.addEventListener('keyup', handleKeyUp)
    
            return () => {
                window.removeEventListener('keydown', handleKeyDown)
                window.removeEventListener('keyup', handleKeyUp)
            }
        }, [])
    
        useFrame((state, delta) => {
            // If the game is over, stop the player's movement
            if (gameState.gameOver) return
    
            const speed = 7
            if (keys.current.left) meshRef.current.position.x -= speed * delta
            if (keys.current.right) meshRef.current.position.x += speed * delta
    
            if (meshRef.current.position.x < -4.5) meshRef.current.position.x = -4.5
            if (meshRef.current.position.x > 4.5) meshRef.current.position.x = 4.5
    
            // Update the global state so obstacles know exactly where the player is
            gameState.playerX = meshRef.current.position.x
        })
    
        return (
            <mesh ref={meshRef} position={[0, 0.5, 0]} castShadow>
                <boxGeometry args={[1, 1, 1]} />
                <meshStandardMaterial color="crimson" />
            </mesh>
        )
    }
    
    function Obstacle({ startZ, startX }) {
        const meshRef = useRef()
    
        useFrame((state, delta) => {
            // Stop moving the obstacles if a collision happened
            if (gameState.gameOver) return
    
            const speed = 15
            meshRef.current.position.z += speed * delta
    
            // When the obstacle passes the camera, reset it and increase the score
            if (meshRef.current.position.z > 5) {
                meshRef.current.position.z = -40
                meshRef.current.position.x = (Math.random() - 0.5) * 8
                gameState.score += 10
            }
    
            // COLLISION DETECTION: Simple mathematical distance check
            // We check the distance between the obstacle and the player on X and Z axes
            const distZ = Math.abs(meshRef.current.position.z - 0) // Player is always at Z = 0
            const distX = Math.abs(meshRef.current.position.x - gameState.playerX)
    
            // If both distances are less than 1 (the size of our cubes), it means they touched
            if (distZ < 1 && distX < 1) {
                gameState.gameOver = true // Boom! End the game
            }
        })
    
        return (
            <mesh ref={meshRef} position={[startX, 0.5, startZ]} castShadow>
                <boxGeometry args={[1, 1, 1]} />
                <meshStandardMaterial color="dodgerblue" />
            </mesh>
        )
    }
    
    function Ground() {
        return (
            <mesh position={[0, -0.05, 0]} receiveShadow>
                <boxGeometry args={[10, 0.1, 40]} />
                <meshStandardMaterial color="#444444" />
            </mesh>
        )
    }
    
    // 2D HTML Overlay component to show the score and Game Over screen
    function UI({ resetGame }) {
        const [score, setScore] = useState(0)
        const [gameOver, setGameOver] = useState(false)
    
        // Use a simple interval to update the React UI with the current game state
        useEffect(() => {
            const interval = setInterval(() => {
                setScore(gameState.score)
                setGameOver(gameState.gameOver)
            }, 100)
            return () => clearInterval(interval)
        }, [])
    
        return (
            <div style={{ position: 'absolute', top: 20, left: 20, color: 'white', fontFamily: 'sans-serif', zIndex: 10 }}>
                <h2>Score: {score}</h2>
    
                {gameOver && (
                    <div style={{
                        position: 'fixed', top: '50%', left: '50%', transform: 'translate(-50%, -50%)',
                        textAlign: 'center', background: 'rgba(0,0,0,0.85)', padding: '40px', borderRadius: '15px'
                    }}>
                        <h1 style={{ color: 'crimson', fontSize: '48px', margin: '0 0 20px 0' }}>GAME OVER</h1>
                        <button onClick={resetGame} style={{
                            padding: '15px 30px', fontSize: '20px', cursor: 'pointer',
                            background: 'white', border: 'none', borderRadius: '8px', fontWeight: 'bold'
                        }}>
                            Restart Game
                        </button>
                    </div>
                )}
            </div>
        )
    }
    
    function App() {
        // We use a React key to completely remount the 3D Canvas when restarting the game
        const [gameKey, setGameKey] = useState(0)
    
        const resetGame = () => {
            gameState.gameOver = false
            gameState.score = 0
            gameState.playerX = 0
            setGameKey(prev => prev + 1)
        }
    
        return (
            <div style={{ width: '100vw', height: '100vh', position: 'relative' }}>
    
                {/* 2D UI Layer sits on top of the 3D Canvas */}
                <UI resetGame={resetGame} />
    
                {/* 3D Game World */}
                <Canvas key={gameKey} shadows camera={{ position: [0, 5, 10], fov: 60 }}>
                    <ambientLight intensity={0.4} />
                    <directionalLight
                        position={[5, 10, 5]}
                        intensity={1.2}
                        castShadow
                        shadow-mapSize={[1024, 1024]}
                    />
    
                    <Player />
                    <Ground />
    
                    <Obstacle startX={0} startZ={-10} />
                    <Obstacle startX={-3} startZ={-20} />
                    <Obstacle startX={3} startZ={-30} />
                    <Obstacle startX={-1} startZ={-40} />
                </Canvas>
            </div>
        )
    }
    
    export default App
    
        
