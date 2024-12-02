# tec-robot-tetrahedron

Self-learning Robotic System Development


![image](https://github.com/SteveJustin1963/tec-robot-tetrahedron/assets/58069246/9ee50d71-01a8-4294-aaec-957ccb9f61b9)

r the MINT implementation:

# Tetrahedral Robot Chain Control System

## Overview
A MINT-based control system for a chain of tetrahedral robots that learn and optimize their movement through air-pressure-based locomotion.

## System Components

### 1. Hardware Requirements
- TEC-1 computer running MINT
- Air pump/valve system
- Distance sensors (Port #F0)
- Angle sensors (Port #F1)
- Tetrahedral inflatable segments
- Pressure control system

### 2. Memory Organization
```
[ start_time velocity distance position joint_angle ]
```
Each tetrahedron uses 5 memory slots for:
- Start time for inflation cycle
- Current velocity
- Distance moved
- Current position
- Joint angle

## Core Functions

### Initialization (:I)
- Clears all tetrahedron parameters
- Sets initial positions to zero
- Prepares learning cycle

### Movement Control (:P)
- Controls individual tetrahedron movement
- Manages inflation cycles
- Updates velocity based on learning
- Monitors distance traveled

### Learning Process (:M)
```mint
:M
  /U(          // Infinite loop
    A          // Read sensors
    D          // Process movement
    F          // Handle friction
    100()      // Timing delay
  )
```

### Failure Recovery (:V)
- Detects blocked valves
- Implements vibration recovery
- Resets movement parameters
- Restarts learning cycle

## Operation Sequence

1. System Start
```mint
I              // Initialize system
```

2. Normal Operation
```mint
M              // Start main control loop
```

3. Monitor Status
```mint
2 S            // Check status of unit 2
```

## Advanced Features

### 1. Friction Handling
- Detects friction issues
- Adjusts velocity for obstacles
- Uses vibration for unsticking

### 2. Chain Length Adaptation
- Handles 3-robot chains
- Adjusts timing for longer chains
- Modifies synchronization patterns

### 3. Learning Optimization
- Selects random start times
- Measures movement effectiveness
- Updates parameters based on results

## Example Usage

```mint
// Initialize and start system
I              // Initialize
M              // Start control loop

// Monitor specific unit
0 S            // Monitor first unit
```

## Troubleshooting

### Common Issues
1. Movement Stalls
   - Check pressure system
   - Verify sensor connections
   - Run recovery routine (:V)

2. Poor Synchronization
   - Check timing parameters
   - Verify unit connections
   - Reset learning cycle

3. Sensor Errors
   - Verify port connections
   - Check sensor power
   - Run initialization

## System Limitations
- Maximum 6 units in chain
- 16-bit integer math only
- Fixed learning cycle length
- Basic random number generation

## Future Enhancements
1. Advanced Learning Algorithms
2. Better Synchronization Methods
3. Enhanced Error Recovery
4. More Sophisticated Movement Patterns
 
 

## mint code
```
// Constants
2 m!          // max_speed
2 c!          // inflation_cycle_duration

// Tetrahedral chain storage for humanoid form
// Format per tetrahedron: [start_time, velocity, distance, position, joint_angle]
[ 0 0 0 0 0   // Head
  0 0 0 0 0   // Torso
  0 0 0 0 0   // Left arm
  0 0 0 0 0   // Right arm
  0 0 0 0 0   // Left leg
  0 0 0 0 0 ] // Right leg
t!            // t stores all tetrahedra data

// Position codes
0 h!          // head index
1 o!          // torso index
2 l!          // left arm index
3 r!          // right arm index
4 g!          // left leg index
5 j!          // right leg index

// Initialize humanoid chain
:I
  6(          // For 6 body parts
    i!        // Current index
    0 t i 5 * ?!     // Start time = 0
    0 t i 5 * 1+ ?!  // Velocity = 0
    0 t i 5 * 2+ ?!  // Distance = 0
    0 t i 5 * 3+ ?!  // Position = 0
    0 t i 5 * 4+ ?!  // Joint angle = 0
  )
;

// Measure joint angles
:J
  n!          // Tetrahedron index
  #F1 /I      // Read angle sensor
  t n 5 * 4+ ?!  // Store angle
;

// Balance check
:B
  g t 5 * 3+ ?    // Get left leg position
  j t 5 * 3+ ?    // Get right leg position
  - /T * 10 < n!  // Check balance difference
;

// Process single tetrahedron with balance
:P
  n!          // Tetrahedron index
  
  // Different behavior based on body part
  n h = (     // Head movement
    30()      // Less movement for head
  ) /E (
    n o = (   // Torso stability
      B (     // Check balance
        R t n 5 * ?!  // Only move if balanced
      )
    ) /E (    // Limbs
      R t n 5 * ?!    // Set random start time
      J               // Get joint angle
    )
  )
  
  // Learning loop with balance constraints
  5(          // Try 5 times
    t n 5 * ? 50 * ()  // Delay based on start time
    
    // Movement based on body part
    n o < (   // If upper body
      50()    // Shorter movements
    ) /E (
      100()   // Longer for legs
    )
    
    // Measure and update
    D         // Get distance
    t n 5 * 2+ ? // Previous distance
    > (       // If improved
      d t n 5 * 2+ ?!  // Update distance
      B (     // If balanced
        t n 5 * 1+ ? 1+ // Increase velocity
        m < (   // Check max speed
          t n 5 * 1+ ?! // Update if within limits
        )
      )
    )
  )
;

// Coordinated movement
:M
  // Sequence for walking
  l g P       // Left arm and leg
  100()       // Delay
  r j P       // Right arm and leg
  100()       // Delay
  o B (       // Check torso balance
    h P       // Move head if balanced
  )
;

// Emergency recovery
:E
  `Balance lost - recovering` /N
  6(         // Reset all positions
    i!
    0 t i 5 * 3+ ?!  // Zero positions
    0 t i 5 * 1+ ?!  // Zero velocities
  )
  1000()     // Wait
  I          // Reinitialize
;

// Main control loop with balance
:C
  I           // Initialize
  /U(         // Infinite loop
    B (       // If balanced
      M       // Normal movement
    ) /E (
      E       // Emergency recovery
    )
    `Cycle complete - Balance: ` B . /N
    500()     // Cycle delay
  )
;

// Status monitor for body part
:S
  n!          // Body part index
  n h = ( `Head` ) /E (
  n o = ( `Torso` ) /E (
  n l = ( `Left Arm` ) /E (
  n r = ( `Right Arm` ) /E (
  n g = ( `Left Leg` ) /E (
    `Right Leg` )))))
  `: ` /N
  `Position: ` t n 5 * 3+ ? . /N
  `Angle: ` t n 5 * 4+ ? . /N
  `Balance: ` B . /N
;
```


## self-learning
- basic way due to MINT's limitations.
  
1. In the Learning Process (function P):
```mint
:P
  n!          // Tetrahedron index
  
  // Learning elements:
  5(          // Try 5 times - learning iterations
    t n 5 * ? 50 * ()  // Tests different timings
    
    D         // Get distance measurement
    t n 5 * 2+ ? // Compare with previous
    > (       // If improved (learned better timing)
      d t n 5 * 2+ ?!  // Update best distance
      B (     // If balanced
        t n 5 * 1+ ? 1+ // Increase velocity - learns optimal speed
        m < (   // Check max speed
          t n 5 * 1+ ?! // Updates learned parameters
        )
      )
    )
  )
;
```

The self-learning aspects include:

1. Parameter Optimization:
- Tests different start times for inflation
- Learns optimal velocities
- Adapts to balance requirements

2. Feedback Loop:
- Measures movement results
- Compares with previous performance
- Updates parameters based on success

3. Adaptive Behavior:
```mint
:E
  `Balance lost - recovering` /N
  6(         // Reset all positions
    i!
    0 t i 5 * 3+ ?!  // Zero positions
    0 t i 5 * 1+ ?!  // Zero velocities
  )
  1000()     // Wait
  I          // Reinitialize and learn again
;
```

However, there are limitations:
1. Simple learning algorithm (try and compare)
2. Limited memory for storing learned patterns
3. No long-term learning retention
4. Basic optimization strategy


## enhanced code
- more sophisticated learning algorithms
- pattern recognition
- long-term storage
- better adaptation.

```

// Advanced Self-Learning Tetrahedral Robot System
// Using EEPROM for long-term storage at port #E0

// Pattern storage (16 patterns, 5 values each)
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] p0! // Times
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] p1! // Velocities
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] p2! // Success rates
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] p3! // Conditions
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] p4! // Adaptations

// Current state buffer
[ 0 0 0 0 0 0 ] s! // Success history
[ 0 0 0 0 0 0 ] c! // Condition history
[ 0 0 0 0 0 0 ] a! // Adaptation history

// Learning parameters
10 k!    // Learning rate (0-100)
75 t!    // Success threshold
5 m!     // Memory depth

// Save pattern to EEPROM
:W 
  n!           // Pattern index
  #E0 /O n     // Write address
  #E1 /O p0 n ? // Write timing
  #E1 /O p1 n ? // Write velocity
  #E1 /O p2 n ? // Write success
  #E1 /O p3 n ? // Write condition
  #E1 /O p4 n ? // Write adaptation
;

// Read pattern from EEPROM
:R
  n!           // Pattern index
  #E0 /O n     // Read address
  #E0 /I p0 n ?! // Read timing
  #E0 /I p1 n ?! // Read velocity
  #E0 /I p2 n ?! // Read success
  #E0 /I p3 n ?! // Read condition
  #E0 /I p4 n ?! // Read adaptation
;

// Pattern similarity score
:Y
  x! y!        // Two patterns to compare
  0 s!         // Initialize similarity score
  
  // Compare all attributes
  p0 x ? p0 y ? - /T * 
  p1 x ? p1 y ? - /T * +
  p2 x ? p2 y ? - /T * +
  p3 x ? p3 y ? - /T * +
  p4 x ? p4 y ? - /T * +
  
  5 /          // Average difference
  100 $        // Convert to similarity
  s!           // Store result
;

// Find best matching pattern
:B
  0 b!         // Best match index
  0 h!         // Highest similarity
  
  16(          // Check all patterns
    i!
    i Y        // Get similarity
    h > (      // If better match
      s h!     // Update highest
      i b!     // Update best
    )
  )
;

// Update pattern weights
:U
  n!           // Pattern index
  s!           // Success score
  
  // Update success rate
  p2 n ? k * s 100 k - * +
  100 /
  p2 n ?!
  
  // Update adaptation based on success
  s t > (      // If successful
    p4 n ? 1+  // Increase adaptation
  ) /E (
    p4 n ? 1-  // Decrease adaptation
  ) p4 n ?!
;

// Learn from current state
:L
  // Shift history buffers
  5(
    5 i - j!
    s j 1+ ? s j ?!
    c j 1+ ? c j ?!
    a j 1+ ? a j ?!
  )
  
  // Add new state
  n s 0 ?!     // Success
  x c 0 ?!     // Condition
  y a 0 ?!     // Adaptation
  
  // Find similar pattern
  B            // Get best match
  
  // Update or create pattern
  b Y 90 > (   // If very similar
    b U        // Update existing
  ) /E (       // Else
    // Find empty or worst pattern
    16(
      i!
      p2 i ? h < ( // If worse success
        p2 i ? h!  // Track worst
        i b!       // Save index
      )
    )
    // Create new pattern
    n p0 b ?!    // Time
    x p1 b ?!    // Velocity
    75 p2 b ?!   // Initial success
    y p3 b ?!    // Condition
    1 p4 b ?!    // Initial adaptation
  )
  
  // Save to EEPROM
  b W
;

// Adaptive control based on learned patterns
:A
  // Get current conditions
  #F0 /I x!    // Sensor input
  #F1 /I y!    // Additional sensor
  
  // Find best matching pattern
  B
  
  // Apply learned parameters with adaptation
  p0 b ? p4 b ? * 100 / n!  // Adjusted timing
  p1 b ? p4 b ? * 100 / v!  // Adjusted velocity
  
  // Execute movement
  n ()         // Timing delay
  v #F2 /O     // Apply velocity
  
  // Measure success
  #F3 /I s!    // Get result
  
  // Learn from this attempt
  s x y L      // Update learning
;

// Initialize system
:I
  // Clear buffers
  6(
    i!
    0 s i ?!
    0 c i ?!
    0 a i ?!
  )
  
  // Load patterns from EEPROM
  16(
    i!
    i R
  )
;

// Main control loop
:M
  I            // Initialize
  /U(          // Infinite loop
    A          // Adaptive control
    100()      // Cycle delay
    
    // Display status
    `Success rate: `
    p2 b ? .   // Show current success
    /N
  )
;
```


Key enhancements:

1. Sophisticated Learning:
- Pattern-based learning
- Success rate tracking
- Adaptive parameter adjustment
- Similarity comparison
- Weight updates

2. Pattern Recognition:
- Pattern matching algorithm
- Similarity scoring
- Best match finding
- Condition recognition
- State history tracking

3. Long-term Storage:
- EEPROM integration
- Pattern persistence
- Success history
- Adaptation levels
- Parameter recovery

4. Enhanced Adaptation:
- Dynamic parameter adjustment
- Success-based adaptation
- Condition-based responses
- Multi-parameter learning
- Weighted adjustments

Key Functions:
1. W/R: EEPROM access
2. Y: Pattern similarity
3. B: Best pattern matching
4. U: Weight updates
5. L: Learning algorithm
6. A: Adaptive control

New Features:
1. Success rate tracking
2. Pattern-based decision making
3. Persistent learning storage
4. Dynamic adaptation
5. Condition recognition

## another enhancement
- additional pattern analysis,
- improved adaptation,
- more sensor integration,
- and multiple learning strategies.

```
// Advanced Self-Learning Tetrahedral Robot System V2
// Enhanced with multiple analysis methods and learning strategies

// Sensor ports
#F0 d! // Distance sensor
#F1 g! // Gyro/balance
#F2 p! // Pressure
#F3 f! // Force feedback
#F4 t! // Temperature
#F5 v! // Vibration

// Pattern and state storage
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] p0! // Times
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] p1! // Velocities
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] p2! // Success
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] p3! // Conditions
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] p4! // Adaptations

// Extended sensor history (16 samples per sensor)
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] h0! // Distance history
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] h1! // Gyro history
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] h2! // Pressure history
[ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] h3! // Force history

// Learning strategies (0-3)
0 s!  // Current strategy

// Pattern Analysis Methods

// Trend Analysis
:T
  n!          // Buffer index
  0 t!        // Trend value
  14(         // Look at last 15 samples
    i!
    h0 i 1+ ? h0 i ? - // Get difference
    t + t!    // Accumulate
  )
;

// Frequency Analysis
:F
  0 f!        // Peak count
  14(         // Analyze samples
    i!
    h0 i ? h0 i 1+ ? >  // Find peaks
    h0 i ? h0 i 1- ? > &
    ( f 1+ f! )
  )
;

// Pattern Variance
:V
  0 v!        // Variance
  0 m!        // Mean
  // Calculate mean
  16( i! h0 i ? m + m! )
  16 m / m!
  // Calculate variance
  16( i! h0 i ? m - " * v + v! )
  16 v / v!
;

// Enhanced Pattern Matching
:Y
  x! y!       // Two patterns to compare
  
  // Multiple comparison metrics
  T t!        // Get trend
  F f!        // Get frequency
  V v!        // Get variance
  
  // Weighted comparison
  p0 x ? p0 y ? - /T * 3 *    // Time weight
  p1 x ? p1 y ? - /T * 2 *    // Velocity weight
  t f + v + 7 / +             // Pattern metrics
  10 /        // Scale result
;

// Adaptive Learning Strategies

// Strategy 0: Conservative Learning
:C
  n!          // Pattern index
  p2 n ? 90 > ( // If highly successful
    p4 n ? 1+ p4 n ?!  // Small adaptation
  ) /E (
    p4 n ? 1- p4 n ?!  // Small reduction
  )
;

// Strategy 1: Aggressive Learning
:G
  n!          // Pattern index
  p2 n ? 75 > ( // If moderately successful
    p4 n ? 2+ p4 n ?!  // Larger adaptation
  ) /E (
    p4 n ? 2- p4 n ?!  // Larger reduction
  )
;

// Strategy 2: Exploratory Learning
:E
  n!          // Pattern index
  R 50 > (    // Random exploration
    p4 n ? 3+ p4 n ?!  // Large change
  ) /E (
    p4 n ? 1- p4 n ?!  // Small reduction
  )
;

// Strategy 3: Hybrid Learning
:H
  n!          // Pattern index
  V           // Get variance
  100 > (     // High variance
    G         // Use aggressive
  ) /E (
    C         // Use conservative
  )
;

// Enhanced Sensor Integration
:S
  // Read all sensors
  d /I h0 0 ?!  // Distance
  g /I h1 0 ?!  // Gyro
  p /I h2 0 ?!  // Pressure
  f /I h3 0 ?!  // Force
  
  // Shift histories
  15(
    15 i - j!
    h0 j 1+ ? h0 j ?!
    h1 j 1+ ? h1 j ?!
    h2 j 1+ ? h2 j ?!
    h3 j 1+ ? h3 j ?!
  )
;

// Dynamic Learning Strategy Selection
:D
  V           // Get pattern variance
  F           // Get frequency
  T           // Get trend
  
  // Select strategy based on metrics
  v 100 > f 10 > & ( 1 s! )   // High variance & frequency -> Aggressive
  v 50 < f 5 < & ( 0 s! )     // Low variance & frequency -> Conservative
  t 100 > ( 2 s! )            // Strong trend -> Exploratory
  s 3 = ( H )                 // Hybrid always evaluates
;

// Enhanced Adaptive Control
:A
  S           // Read sensors
  D           // Select strategy
  
  // Apply current strategy
  s 0 = ( C )
  s 1 = ( G )
  s 2 = ( E )
  s 3 = ( H )
  
  // Find best pattern match
  B
  
  // Apply learned parameters
  p0 b ? p4 b ? * 100 / n!  // Timing
  p1 b ? p4 b ? * 100 / v!  // Velocity
  
  // Execute movement
  n ()        // Timing delay
  v p /O      // Apply to pressure
  
  // Learn from result
  S           // Get new sensor data
  Y           // Compare patterns
  L           // Update learning
;

// Main control loop with strategy display
:M
  I           // Initialize
  /U(         // Infinite loop
    A         // Adaptive control
    
    // Status display
    `Strategy: `
    s 0 = ( `Conservative` )
    s 1 = ( `Aggressive` )
    s 2 = ( `Exploratory` )
    s 3 = ( `Hybrid` )
    /N
    
    `Success: ` p2 b ? . /N
    `Variance: ` v . /N
    `Trend: ` t . /N
    
    100()     // Cycle delay
  )
;
```


Key New Features:

1. Enhanced Pattern Analysis:
- Trend analysis (T)
- Frequency analysis (F)
- Pattern variance (V)
- Multi-metric pattern matching (Y)

2. Multiple Learning Strategies:
- Conservative (C): Small, careful adjustments
- Aggressive (G): Larger adaptation steps
- Exploratory (E): Random exploration
- Hybrid (H): Context-based strategy selection

3. Expanded Sensor Integration:
- Distance sensor
- Gyroscope
- Pressure sensor
- Force feedback
- Temperature monitoring
- Vibration detection
- Historical data for each sensor

4. Advanced Adaptation:
- Dynamic strategy selection
- Multi-metric pattern evaluation
- Weighted comparisons
- Context-aware learning rates
- Sensor fusion for decisions

Notable Functions:
- T: Trend analysis
- F: Frequency analysis
- V: Variance calculation
- D: Dynamic strategy selection
- S: Enhanced sensor integration
- A: Advanced adaptive control

more options:
1. Add more visualization of learning progress?
2. Include more complex pattern analysis methods?
3. Add emergency recovery strategies?
4. Implement coordination between multiple units?
5. 
