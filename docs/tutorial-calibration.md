# Calibration Tutorial

This document explains how calibration is implemented in libsurvive, specifically for a setup with 2 HTC Vive Base Station 2.0 and 1 HTC Vive Tracker 3.0.

## Overview

Calibration is the process of determining the positions and orientations of lighthouse base stations in relation to the tracked objects. This is a critical step that enables accurate 6DOF tracking. The calibration process involves several stages:

1. **OOTX Data Collection**: Gathering calibration parameters from base stations
2. **Light Data Processing**: Processing sensor activation data from trackers
3. **Pose Solving**: Computing relative positions between base stations and trackers
4. **World Coordinate System Establishment**: Converting relative positions to a fixed world coordinate system
5. **Configuration Persistence**: Saving calibration results for future use

## User Guide: How to Perform Calibration

This section provides step-by-step instructions for performing calibration with your HTC Vive Base Station 2.0 and Tracker 3.0 setup.

### System Limits

**⚠️ Important Limitations:**

- **Maximum Base Stations**: libsurvive supports up to **16 base stations** (defined by `NUM_GEN2_LIGHTHOUSES`). This limit applies to Base Station 2.0 (Lighthouse 2.0) systems. For Base Station 1.0 systems, the limit is 2 base stations.
- **Maximum Trackers**: There is no hard-coded limit on the number of trackers, but practical limits depend on:
  - USB bandwidth and port availability
  - CPU processing power
  - System memory
  - In practice, 10-20 trackers are commonly used, but more are possible with sufficient hardware

**Note**: If you exceed the maximum number of base stations, the system will only recognize and calibrate the first 16 base stations it detects. Additional base stations will be ignored.

### Prerequisites

Before starting calibration, ensure you have:

- 2-16 × HTC Vive Base Station 2.0 powered on and in tracking mode (or 2 × Base Station 1.0)
- 1 or more × HTC Vive Tracker 3.0 (or other compatible tracked devices) connected to your computer via USB
- libsurvive installed and built (see main README.md for installation instructions)
- SteamVR closed (libsurvive may compete for device access with SteamVR)

### Recommended Room Sizes for Different Base Station Configurations

**Base Station 2.0 Tracking Range:**
- **Effective Range**: Up to 7 meters (23 feet) from base station to tracker
- **Optimal Range**: 2-5 meters (6.5-16 feet) for best tracking quality
- **Field of View**: Approximately 120 degrees per base station

**Recommended Room Sizes:**

| Base Stations | Minimum Room Size | Maximum Room Size | Typical Use Case |
|--------------|-------------------|-------------------|------------------|
| **2 Base Stations** | 2m × 1.5m (6.5' × 5') | 7m × 7m (23' × 23') | Small to medium rooms, residential VR setups |
| **3-4 Base Stations** | 3m × 3m (10' × 10') | 10m × 10m (33' × 33') | Large rooms, professional VR setups, arcades |
| **5-8 Base Stations** | 5m × 5m (16' × 16') | 15m × 15m (50' × 50') | Very large spaces, warehouses, event venues |
| **9-16 Base Stations** | 10m × 10m (33' × 33') | 20m+ × 20m+ (65'+ × 65'+) | Industrial applications, research labs, motion capture studios |

**Important Considerations:**
- **Tracking Quality**: More base stations provide better tracking coverage, especially at the edges of the tracking volume
- **Occlusion**: Additional base stations help reduce occlusion when users or objects block line of sight
- **Redundancy**: More base stations provide redundancy if one fails or is temporarily blocked
- **Calibration Complexity**: More base stations require more careful calibration, especially in large spaces

**Distance Between Base Stations:**
- **Minimum**: 2 meters (6.5 feet) between base stations
- **Maximum**: 10 meters (33 feet) for optimal coverage overlap
- **Recommended**: 3-5 meters (10-16 feet) apart for balanced coverage

### Multiple-Room Setup

**Can you use base stations across multiple rooms?**

**Short Answer**: Technically possible but not recommended for a single unified tracking space.

**Detailed Explanation:**

1. **Single Unified Tracking Space (Not Recommended)**:
   - Base stations are designed to work within a single, continuous tracking volume
   - Walls and obstacles between rooms will block line of sight, causing tracking failures
   - Tracking quality will degrade significantly when moving between rooms
   - Calibration becomes very difficult as trackers cannot see all base stations simultaneously

2. **Separate Rooms with Independent Tracking Spaces (Possible)**:
   - **Setup**: Each room has its own set of base stations (2-4 per room recommended)
   - **Configuration**: Each room maintains its own separate `config.json` file
   - **Switching**: Users must recalibrate or reload configuration when moving between rooms
   - **Use Case**: Different applications or users in different rooms
   - **Limitation**: Trackers cannot seamlessly move between rooms without recalibration

3. **Large Open Space with Multiple Zones (Recommended)**:
   - **Setup**: Use 4-8 base stations distributed throughout a large open space
   - **Configuration**: Single unified tracking space with all base stations calibrated together
   - **Advantage**: Seamless tracking across the entire space
   - **Use Case**: Large warehouses, event halls, or open commercial spaces

**Best Practices for Multi-Room Scenarios:**

- **Option A - Separate Configurations**: 
  - Maintain separate calibration files for each room
  - Use libsurvive instances with different config files: `./bin/survive-cli --config <room_config.json>`
  - Manually switch configurations when moving between rooms

- **Option B - Large Open Space**:
  - Remove or minimize walls/obstacles between areas
  - Install base stations high enough to see over obstacles
  - Use more base stations (6-12) for better coverage
  - Ensure all base stations can see each other or have overlapping coverage

- **Option C - Hybrid Approach**:
  - Use 4-6 base stations in a central "hub" area with good coverage
  - Add 2 base stations in adjacent rooms for partial coverage
  - Accept that tracking quality may be reduced in peripheral rooms

**Technical Limitations:**

- **Line of Sight**: Base stations require clear line of sight to trackers for accurate tracking
- **Calibration**: All base stations must be visible from at least one calibration point
- **Synchronization**: Base stations need to synchronize with each other (automatic for Gen 2)
- **Coordinate System**: A single world coordinate system is established during calibration

**Multi-Room Setup with High Mounting (No Ceiling or High Ceilings):**

If your setup has **no ceiling** or **very high ceilings**, you can mount base stations **higher than the walls** to create a unified tracking space across multiple rooms:

**Advantages:**
- **Unified Tracking Space**: Base stations mounted above wall height can see across multiple rooms
- **Seamless Movement**: Trackers can move between rooms without recalibration
- **Single Coordinate System**: All rooms share the same world coordinate system
- **Better Coverage**: Base stations can cover larger areas by being positioned above obstacles

**Setup Requirements:**
- **Mounting Height**: Base stations must be mounted **at least 0.5-1 meter (1.5-3 feet) higher than the tallest wall** to ensure clear line of sight
- **Installation**: Mount base stations on tall poles, building structures, or high mounting brackets
- **Recommended Height**: 3-5 meters (10-16 feet) or higher, depending on wall height
- **Angling**: Angle base stations downward at 30-45 degrees to cover the tracking volume below
- **Distribution**: Space base stations evenly to ensure coverage overlap across all rooms

**Configuration:**
1. **Initial Setup**: Mount all base stations high enough to see over walls
2. **Calibration**: Place tracker(s) in a central location where they can see all base stations
3. **Testing**: Move trackers between rooms to verify tracking quality
4. **Adjustment**: If tracking is poor in certain areas, add more base stations or adjust mounting positions

**Best Practices:**
- **Height Safety**: Ensure base stations are securely mounted and accessible for maintenance
- **Coverage Testing**: Test tracking quality in each room, especially near walls
- **Power Supply**: Consider power distribution for base stations at high mounting points
- **Maintenance**: Ensure base stations are accessible for firmware updates and troubleshooting

**Example Scenarios:**
- **Open Warehouse**: Multiple sections separated by partitions, base stations mounted on ceiling beams
- **High-Ceiling Hall**: Multiple rooms with partial walls, base stations mounted above wall height
- **Outdoor/Indoor Mixed**: Large space with partial walls, base stations on tall poles

**Limitations:**
- **Wall Height**: Very tall walls (>3-4 meters) may still block line of sight
- **Obstacles**: Furniture, equipment, or other tall objects may still cause occlusion
- **Calibration Difficulty**: May require climbing or using lifts to access base stations during calibration
- **Power/Cable Management**: More complex wiring for high-mounted base stations

### Step 1: Setup Your Hardware

1. **Mount Base Stations**: 
   - **For 2 Base Stations**: Position them in opposite corners of your tracking area
   - **For 3-4 Base Stations**: Use a square or rectangular pattern at the corners
   - **For 5-8 Base Stations**: Distribute evenly around your tracking volume for optimal coverage
   - **For 9-16 Base Stations**: Create a grid pattern covering the entire tracking volume
   - Mount all base stations at a height of 2-3 meters (6-10 feet), angled downward at approximately 30-45 degrees
   - Ensure they have clear line of sight to each other and to the tracking volume
   - For large spaces, ensure base stations are spaced 3-5 meters apart for optimal coverage overlap
   - Base Station 2.0 units will automatically assign channels (B, C, D, etc.) for proper synchronization
   - **Important**: Do not exceed 16 base stations - the system will only recognize the first 16

2. **Power On Base Stations**:
   - Connect all base stations to power
   - Wait for them to fully initialize (LED indicators will show their status)
   - Ensure all base stations are in tracking mode (not standby)
   - Verify that each base station has a unique channel assignment (visible in LED indicators)

3. **Connect Trackers**:
   - Connect your Vive Tracker 3.0 (or multiple trackers) to your computer via USB
   - Each tracker should power on automatically
   - The system will automatically detect all connected trackers
   - If you have multiple trackers, you can connect them all at once - calibration will use the best available tracker for each base station

### Step 2: Initial Calibration

1. **Start libsurvive**:
   ```bash
   ./bin/survive-cli
   ```
   
   Or if you want to see more detailed output:
   ```bash
   ./bin/survive-cli --v 100
   ```

2. **Position the Tracker(s)**:
   - **For 2 Base Stations**: Place the tracker in a location where it can clearly "see" both base stations
   - **For 3+ Base Stations**: Place the tracker in a central location where it can see as many base stations as possible
   - If using multiple trackers, you can keep one or more stationary during calibration
   - The tracker(s) should be placed on a flat, stable surface
   - **⚠️ IMPORTANT: Keep the tracker(s) completely stationary during calibration - NO MOTION IS NEEDED OR RECOMMENDED**
   - The tracker's sensors should have an unobstructed view of the base stations
   - **Tip**: If you have multiple trackers, the system will automatically use the one with the best sensor coverage for calibration

3. **Wait for Calibration**:
   - The system will automatically begin collecting OOTX data from all base stations
   - **Keep the tracker stationary** - do not move it during this phase
   - This typically takes 10-60 seconds per base station depending on signal quality
   - For multiple base stations, calibration may take longer as the system needs to collect OOTX data from each one
   - You should see messages in the console indicating OOTX packets are being received
   - Example: `Got OOTX packet 1 00000001`, `Got OOTX packet 2 00000002`, etc.
   - Each base station will be assigned a unique ID visible in these messages

4. **Monitor Progress**:
   - **Continue to keep the tracker stationary** while monitoring progress
   - Watch the console output for calibration status messages
   - The system will indicate when base station positions are being solved
   - You may see messages like "Using LH 0 (...) as reference lighthouse"
   - For multiple base stations, you'll see progress for each one:
     - `Solving for X correspondents` (where X is the number of sensor measurements)
     - Messages indicating which base stations are being calibrated
   - Calibration is complete when all base stations show `PositionSet = 1` in the logs
   - **For 3+ Base Stations**: If not all base stations calibrate initially:
     - **Only then** move the tracker to a new position where it can see the remaining base stations
     - **Keep it stationary again** at the new position until the remaining base stations calibrate
   - **Note**: Motion is NOT required for calibration - the system only needs the tracker to be stationary to collect sufficient light data

### Why No Motion is Required: Difference from Camera Calibration

You may be familiar with camera calibration, which typically requires sampling multiple points throughout a space (e.g., moving a calibration pattern to different positions). **libsurvive's calibration is fundamentally different** and does not require motion. Here's why:

**Traditional Camera Calibration:**
- Requires **known 3D reference points** (e.g., checkerboard pattern with known dimensions)
- Needs **multiple views** of the same pattern from different positions and orientations
- Solves for camera intrinsic parameters (focal length, distortion) and extrinsic parameters (position, orientation)
- Requires **2D-3D correspondences** - matching image points to known 3D world coordinates

**libsurvive Lighthouse Calibration (Why It's Different):**

1. **Known Sensor Positions on Tracker**:
   - The tracker has **factory-calibrated sensor positions** embedded in its firmware
   - Each sensor's 3D position relative to the tracker is precisely known
   - This eliminates the need for external reference points

2. **Active Illumination System**:
   - Base stations **actively sweep** laser beams across the tracking volume
   - The tracker receives **angle measurements** (not 2D image projections)
   - Each sensor reports the angle at which it received the laser sweep
   - Multiple sensors on a **stationary tracker** provide multiple angle measurements simultaneously

3. **Sufficient Constraints from Single Position**:
   - When a tracker is stationary, **all sensors receive light from the same base station** at the same time
   - Each sensor provides **two angle measurements** (one for X-axis sweep, one for Y-axis sweep)
   - With ~20-30 sensors on a typical tracker, this provides **40-60 angle measurements** per base station
   - These measurements are **sufficient to solve** for the base station's position and orientation relative to the tracker

4. **Mathematical Sufficiency**:
   - The problem is: Given N known sensor positions and N angle measurements, solve for the base station pose
   - This is a **Perspective-n-Point (PnP) problem** variant
   - With 4-5 sensors, the problem becomes solvable; with 20+ sensors, it's highly overdetermined
   - **No motion is needed** because a single stationary position provides enough constraints

5. **World Coordinate System Establishment**:
   - The IMU (accelerometer) provides **gravity direction** to establish "up"
   - The first base station establishes the **reference direction**
   - Together, these create a consistent world coordinate system without needing multiple positions

**Why Motion is Actually Detrimental:**
- Moving the tracker during calibration **introduces uncertainty** about the tracker's position
- The calibration algorithm assumes the tracker is stationary to accurately correlate sensor measurements
- Motion would require solving for **both** the tracker trajectory **and** base station positions simultaneously, which is more complex and less accurate
- Stationary data allows the system to **accumulate many measurements** from the same position, improving accuracy

**Comparison Summary:**

| Aspect | Camera Calibration | libsurvive Calibration |
|--------|-------------------|----------------------|
| **Reference Points** | External (checkerboard, known 3D points) | Internal (sensor positions on tracker) |
| **Motion Required** | Yes (multiple positions) | No (stationary) |
| **Measurement Type** | 2D image coordinates | 3D angle measurements |
| **Illumination** | Passive (ambient light) | Active (laser sweeps) |
| **Constraints per Position** | Limited (one view) | Many (all sensors simultaneously) |
| **Coordinate System** | Established from reference points | Established from gravity + first base station |

**In Summary**: libsurvive's calibration works because the tracker's sensor positions are known, and multiple sensors on a stationary tracker provide enough angle measurements to solve for base station positions. This is fundamentally different from camera calibration, which requires external reference points and multiple views.

### Step 3: Verify Calibration

After calibration completes, verify that it was successful:

1. **Check Configuration File**:
   - The calibration data is saved to `config.json` in `XDG_CONFIG_HOME/libsurvive` (typically `~/.config/libsurvive/config.json` on Linux)
   - Open the file and verify that **all** base stations have:
     - `"OOTXSet": 1` (OOTX data collected)
     - `"PositionSet": 1` (position solved)
     - Valid `"pose"` arrays with 7 values (position x,y,z and rotation quaternion w,x,y,z)
     - Unique `"id"` values for each base station
   - **For Multiple Base Stations**: Count the number of base station entries in the config file - it should match the number of base stations you have (up to 16)
   - **For Multiple Trackers**: The configuration file contains base station positions only - tracker-specific data is stored separately

2. **Test Tracking**:
   - Move the tracker(s) around the tracking volume
   - The system should now provide accurate 6DOF pose data for all trackers
   - **With Multiple Base Stations**: You should have better tracking coverage, especially at the edges of your tracking volume
   - **With Multiple Trackers**: All trackers should be tracked simultaneously
   - Use the visualization tool to verify tracking:
     ```bash
     ./bin/survive-websocketd & xdg-open ./tools/viz/index.html
     ```
   - The visualization will show all connected trackers and all calibrated base stations

### Step 4: How to Change Base Station Channel

Base Station 2.0 supports channels 1-16 (internally represented as modes B, C, D, etc.). By default, base stations automatically assign channels to avoid conflicts. However, you may want to manually set specific channels for your setup.

**Prerequisites:**
- Python 3 with `bluepy` library installed: `pip3 install bluepy`
- Linux system with Bluetooth support (required for GATT communication)
- Root access or proper Bluetooth permissions

**Method 1: Using bsd_ctrl.py Tool**

libsurvive includes a tool for managing base station channels:

1. **Navigate to the tool directory**:
   ```bash
   cd tools/bsd_ctrl
   ```

2. **View current base station channels**:
   ```bash
   python3 bsd_ctrl.py -t 4
   ```
   This scans for base stations and displays their current mode/channel assignments.

3. **Set a specific channel for a base station**:
   ```bash
   python3 bsd_ctrl.py -m 2 -d <MAC_ADDRESS>
   ```
   Replace `<MAC_ADDRESS>` with the Bluetooth MAC address of the base station you want to configure. The `-m` parameter sets the mode (1-16).

4. **Auto-assign channels to resolve conflicts**:
   ```bash
   python3 bsd_ctrl.py -t 4
   ```
   If multiple base stations have the same channel, the tool will automatically reassign them to unique channels.

**Method 2: Using libsurvive's GATT Driver**

libsurvive's GATT driver can automatically resolve channel conflicts when enabled:

1. **Enable GATT driver**:
   ```bash
   ./bin/survive-cli --gatt
   ```
   The GATT driver will detect base stations via Bluetooth and automatically reassign channels if conflicts are detected.

**Important Notes:**

- **Channel Range**: Base Station 2.0 supports channels 1-16 (mapped to modes B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q respectively)
- **Persistence**: Channel changes are saved to the base station and persist across power cycles
- **LED Indicators**: The base station LED indicators will show the current channel assignment
- **Power Cycling**: After changing channels, you may need to power cycle the base station for the change to take full effect
- **Verification**: After changing channels, verify the new channel assignment using `bsd_ctrl.py` or check the libsurvive logs

**Troubleshooting Channel Issues:**

- **Cannot connect to base station**: Ensure Bluetooth is enabled and you have proper permissions
- **Channel change not persisting**: Try power cycling the base station after changing the channel
- **Multiple base stations on same channel**: Use `bsd_ctrl.py` without the `-m` flag to auto-assign unique channels
- **Channel conflicts during tracking**: libsurvive's GATT driver will automatically resolve conflicts if enabled

### Step 5: Force Recalibration (if needed)

If you need to recalibrate (e.g., base stations were moved), you have two options:

**Option 1: Use the force-calibrate flag** (recommended):
```bash
./bin/survive-cli --force-calibrate
```
This reuses existing OOTX data but recalculates positions, making it faster than a full recalibration.

**Option 2: Delete the configuration file**:
```bash
rm ~/.config/libsurvive/config.json
./bin/survive-cli
```
This forces a complete recalibration from scratch.

### Tips for Best Results

1. **Optimal Placement**:
   - Place the tracker in the center of your tracking volume during calibration
   - Ensure the tracker is at least 1-2 meters away from each base station
   - Avoid reflective surfaces that could cause interference

2. **During Calibration**:
   - **Keep the tracker completely stationary** - NO MOTION IS NEEDED OR REQUIRED
   - The system uses stationary light data to solve for base station positions
   - Keep the tracker stationary for at least 30-60 seconds (or until calibration completes)
   - Don't move the tracker until you see confirmation that calibration is complete
   - The tracker should remain powered and connected during the entire process
   - **Important**: Moving the tracker during calibration can interfere with the position-solving process

3. **Multiple Trackers**:
   - If you have multiple trackers, you only need to calibrate once per base station setup
   - The calibration is stored in the configuration file and shared across all trackers
   - Subsequent trackers will automatically use the existing calibration
   - **Calibration Process with Multiple Trackers**:
     - Connect all trackers to your computer via USB
     - During calibration, the system will use the tracker with the most sensors (typically the HMD) as the primary calibration device
     - If multiple trackers are present, the system will prioritize the one with the best sensor coverage for each base station
     - All trackers can contribute data to the calibration process, improving accuracy
     - Once calibration is complete, all connected trackers will use the same base station positions
   - **Best Practice**: Keep at least one tracker stationary during calibration to establish a stable reference point

4. **Multiple Base Stations (3-16)**:
   - **Setup**: Mount base stations in a pattern that provides good coverage of your tracking volume
     - For 3-4 base stations: Use a square or rectangular pattern at the corners
     - For 5-8 base stations: Add base stations along walls or at midpoints
     - For 9-16 base stations: Distribute evenly around a large tracking volume
   - **Channel Assignment**: Base Station 2.0 automatically assigns channels (modes B, C, D, etc.) to avoid conflicts
     - The system supports channels 0-15 (16 total channels)
     - Each base station will use a unique channel/frequency
   - **Calibration Strategy**:
     - Place the tracker in a central location where it can see as many base stations as possible
     - The system will calibrate all visible base stations simultaneously
     - If not all base stations are visible from one location:
       - First calibrate with the tracker seeing as many base stations as possible
       - Move the tracker to another location to see additional base stations
       - Keep it stationary until the remaining base stations calibrate
     - The reference base station (first calibrated) establishes the world coordinate system
   - **Verification**: Check that all base stations show `PositionSet = 1` in the logs or configuration file
   - **Performance**: More base stations provide better tracking coverage but may slightly increase processing overhead

5. **Large Spaces**:
   - If your tracking space is large and a single tracker can't see all base stations:
     - First calibrate with the tracker seeing as many base stations as possible
     - Then move the tracker to a position where it can see the remaining base stations
     - Keep it stationary again until the remaining base stations calibrate

### Common Issues and Solutions

**Issue: "OOTX not set" messages**
- **Solution**: Ensure base stations are powered on and in tracking mode. Wait longer (up to 60 seconds) for OOTX data to be received. Check that there are no obstructions between the tracker and base stations.

**Issue: "Can't solve for only X points"**
- **Solution**: The tracker needs to see at least 4-5 sensors from each base station. Move the tracker to a better position with clearer line of sight to both base stations.

**Issue: Calibration takes too long**
- **Solution**: Ensure good line of sight between tracker and base stations. Check for reflections or interference. Verify base stations are properly synchronized.

**Issue: Tracking is inaccurate after calibration**
- **Solution**: Force recalibration with `--force-calibrate`. Verify base stations haven't moved. Check that the configuration file contains valid pose data for all base stations.

**Issue: Not all base stations are calibrating (3+ base stations)**
- **Solution**: Ensure the tracker can see all base stations. You may need to move the tracker to different positions to see all base stations. Keep it stationary at each position until the base stations calibrate. Check that you haven't exceeded the 16 base station limit.

**Issue: Only some of my multiple trackers are being tracked**
- **Solution**: Ensure all trackers are properly connected via USB and powered on. Check USB bandwidth - too many devices on one USB controller can cause issues. Try connecting trackers to different USB ports or controllers. Verify that all trackers are detected by running `./bin/survive-cli --v 100` and checking the device list.

**Issue: System is slow with many base stations and/or trackers**
- **Solution**: This is expected with large setups. Consider:
  - Using a more powerful CPU
  - Reducing verbosity (`--v 10` instead of `--v 100`)
  - Closing other applications that use CPU/GPU resources
  - For very large setups (10+ base stations + 10+ trackers), you may need to optimize your system configuration

### Using Visualization During Calibration

For a visual representation of the calibration process, you can use the web-based visualization tool:

```bash
# Terminal 1: Start the websocket server
./bin/survive-websocketd --v 100

# Terminal 2: Open the visualization (or use your browser)
xdg-open ./tools/viz/index.html
```

The visualization will show:
- Base stations (lighthouses) as they are detected and calibrated
- Tracker position and orientation in real-time
- The coordinate system being established

This is particularly helpful for understanding the calibration process and verifying that everything is working correctly.

## Components Involved

### Base Station 2.0 (Lighthouse 2.0)

The Base Station 2.0 uses a rotating laser system with two axes (X and Y) that sweep across the tracking volume. Each base station contains factory calibration data that is transmitted via OOTX (Out-of-TX) protocol.

### Tracker 3.0

The Vive Tracker 3.0 is equipped with multiple photodiodes (sensors) arranged in a known pattern on its surface. When a laser sweep from a base station hits a sensor, the tracker records the timing information, which is used to calculate angles.

## Calibration Process

### Stage 1: OOTX Data Collection

**What is OOTX?**

OOTX (Out-of-TX) is a protocol used by lighthouse base stations to broadcast their factory calibration parameters. This data is embedded in the sync pulses and must be decoded.

**Implementation Details:**

The OOTX decoder collects bits from the sync pulses and reconstructs the calibration packet:

```32:85:src/survive_process_gen2.c
STATIC_CONFIG_ITEM(SERIALIZE_OOTX, "serialize-ootx", 'b', "Serialize out ootx", 0)
static void ootx_packet_clbk_d_gen2(ootx_decoder_context *ct, ootx_packet *packet) {
	SurviveContext *ctx = ((SurviveObject *)(ct->user))->ctx;
	int id = ct->user1;

	lighthouse_info_v15 v15;
	init_lighthouse_info_v15(&v15, packet->data);

	if (survive_configi(ctx, SERIALIZE_OOTX_TAG, SC_GET, 0) == 1) {
		char filename[128];
		snprintf(filename, 128, "LH%02d_%08x.ootx", v15.mode_current & 0x7F, (unsigned)v15.id);
		FILE *f = fopen(filename, "w");
		fwrite(packet->data, packet->length, 1, f);
		fclose(f);
	}

	BaseStationData *b = &ctx->bsd[id];
	b->OOTXChecked |= true;
	FLT accel[3] = {v15.accel_dir[0], v15.accel_dir[1], v15.accel_dir[2]};
	bool upChanged = norm3d(b->accel) != 0.0 && dist3d(b->accel, accel) > 1e-3;

	if (upChanged) {
		SV_VERBOSE(10, "OOTX up direction changed for %x (%f)", b->BaseStationID, norm3d(b->accel));
	}
	bool doSave = b->BaseStationID != v15.id || b->OOTXSet == false || upChanged;
	b->OOTXSet = 1;

	if (doSave) {
	  SV_INFO("Got OOTX packet %d %08x", ctx->bsd[id].mode, (unsigned)v15.id);

		b->BaseStationID = v15.id;
		for (int i = 0; i < 2; i++) {
			b->fcal[i].phase = v15.fcal_phase[i];
			b->fcal[i].tilt = v15.fcal_tilt[i];
			b->fcal[i].curve = v15.fcal_curve[i];
			b->fcal[i].gibpha = v15.fcal_gibphase[i];
			b->fcal[i].gibmag = v15.fcal_gibmag[i];
			b->fcal[i].ogeephase = v15.fcal_ogeephase[i];
			b->fcal[i].ogeemag = v15.fcal_ogeemag[i];
		}

		for (int i = 0; i < 3; i++) {
			b->accel[i] = v15.accel_dir[i];
		}
		b->sys_unlock_count = v15.sys_unlock_count;

		// Although we know this already....
		b->mode = v15.mode_current & 0x7F;

		survive_reset_lighthouse_position(ctx, id);

		SURVIVE_INVOKE_HOOK(ootx_received, ctx, id);
	}
}
```

**Calibration Parameters Extracted:**

For Base Station 2.0, each base station has calibration parameters for two axes (X and Y rotors):

- **phase**: Phase offset for the rotor
- **tilt**: Tilt angle of the rotor plane
- **curve**: Curvature correction parameter
- **gibpha/gibmag**: Gibbous phase and magnitude corrections
- **ogeephase/ogeemag**: Ogee phase and magnitude corrections (Gen 2 specific)
- **accel_dir**: Gravity direction vector (used to establish "up")

These parameters are stored in the `BaseStationCal` structure:

```257:267:include/libsurvive/survive.h
typedef struct BaseStationCal {
	FLT phase;
	FLT tilt;
	FLT curve;
	FLT gibpha;
	FLT gibmag;

	// Gen 2 specific cal params
	FLT ogeephase;
	FLT ogeemag;
} BaseStationCal;
```

**Why OOTX is Important:**

The factory calibration parameters are essential for accurate reprojection. Without these parameters, the system cannot correctly interpret the angle measurements from the laser sweeps, leading to significant tracking errors.

### Stage 2: Light Data Collection

While collecting OOTX data, the tracker also collects light activation data. When a laser sweep hits a sensor on the tracker, it records:

- Which sensor was activated
- Which base station (identified by channel/frequency)
- Which axis (X or Y)
- The timing of the activation

This data is processed to extract angle measurements for each sensor-base station-axis combination.

### Stage 3: Position Solving

Once sufficient light data and OOTX data are collected, the poser (pose solver) attempts to determine the relative positions of the base stations.

**The Problem:**

Given:
- Known sensor positions on the tracker (from device configuration)
- Angle measurements from multiple sensors to each base station
- Base station calibration parameters

Find:
- The position and orientation of each base station relative to the tracker

**Solution Approaches:**

libsurvive uses several poser algorithms:

1. **Barycentric SVD Poser**: A seed poser that works without an initial estimate, used at startup
2. **EPnP Poser**: Based on Efficient Perspective-n-Point algorithm
3. **MPFit Poser**: Uses Levenberg-Marquardt optimization for non-linear least squares

The poser solves for the relative pose of each base station. For a setup with 2 base stations and 1 tracker, the poser will:

1. Solve for Base Station 0 position relative to tracker
2. Solve for Base Station 1 position relative to tracker

**Example from EPnP Poser:**

```62:104:src/poser_epnp.c
static int opencv_solver_fullscene(SurviveObject *so, PoserDataFullScene *pdfs) {
	SurvivePose lh2object[NUM_GEN2_LIGHTHOUSES] = { 0 };
	for (int lh = 0; lh < so->ctx->activeLighthouses; lh++) {
		epnp pnp = {.fu = 1, .fv = 1};
		epnp_set_maximum_number_of_correspondences(&pnp, so->sensor_ct);

		for (size_t i = 0; i < so->sensor_ct; i++) {
			FLT *_ang = pdfs->angles[i][lh];
			if (isnan(_ang[0]) || isnan(_ang[1]))
				continue;

			FLT ang[2];
			survive_apply_bsd_calibration(so->ctx, lh, _ang, ang);

			epnp_add_correspondence(&pnp, so->sensor_locations[i * 3 + 0], so->sensor_locations[i * 3 + 1],
									so->sensor_locations[i * 3 + 2], get_u(ang), get_v(ang));
		}

		SurviveContext *ctx = so->ctx;
		SV_INFO("Solving for %d correspondents", pnp.number_of_correspondences);
		if (pnp.number_of_correspondences <= 4) {
			SV_INFO("Can't solve for only %d points on lh %d\n", pnp.number_of_correspondences, lh);
			continue;
		}

		lh2object[lh] = solve_correspondence(so, &pnp, true);

		epnp_dtor(&pnp);
	}

	PoserData_lighthouse_poses_func(&pdfs->hdr, so, lh2object, so->ctx->activeLighthouses, 0);

	return 0;
}
```

### Stage 4: World Coordinate System Establishment

The positions solved in Stage 3 are relative to the tracker's coordinate system, which is arbitrary. To establish a consistent world coordinate system, libsurvive performs several transformations:

**Step 1: Gravity Alignment**

The system uses IMU (accelerometer) data to determine the "up" direction. This ensures that the Z-axis points upward, regardless of how the tracker is oriented during calibration.

**Step 2: Reference Base Station**

The first base station (or the one specified by `reference-basestation` config) is used as a reference. It is positioned on the X-axis by rotating around the Z-axis.

**Step 3: Coordinate System Normalization**

The world coordinate system is established with:
- Origin: Initially at the tracker position (or optionally at the first base station)
- Z-axis: Points upward (aligned with gravity)
- X-axis: Points toward the reference base station (projected onto XY plane)
- Y-axis: Completes the right-handed coordinate system

**Implementation:**

```142:201:src/poser.c
		// Assume that the space solved for is valid but completely arbitrary. We are going to do a few things:
		// a) Using the gyro data, normalize it so that gravity is pushing straight down along Z
		// c) Assume the object is at origin
		// b) Place the first lighthouse on the X axis by rotating around Z
		//
		// This calibration setup has the benefit that as long as the user is calibrating on the same flat surface,
		// calibration results will be roughly identical between all posers no matter the orientation the object is
		// lying
		// in.
		//
		// We might want to go a step further and affix the first lighthouse in a given pose that preserves up so that
		// it doesn't matter where on that surface the object is.
		bool worldEstablished = quatmagnitude(object_pose->Rot) != 0;
		SurvivePose object2arb = {.Rot = {1.}};
		if (object_pose && !quatiszero(object_pose->Rot))
			object2arb = *object_pose;
		SurvivePose lighthouse2arb = *lighthouse_pose;

		SurvivePose obj2world, lighthouse2world;
		// Purposefully only set this once. It should only depend on the first (calculated) lighthouse
		if (!worldEstablished) {
			bool centerOnLh0 =

			// Start by just moving from whatever arbitrary space into object space.
			SurvivePose arb2object;
			InvertPose(&arb2object, &object2arb);

			SurvivePose lighthouse2obj;
			ApplyPoseToPose(&lighthouse2obj, &arb2object, &lighthouse2arb);
			SurvivePose arb2world = arb2object;

			// Find the poses that map to the above

			// Find the space with the same origin, but rotated so that gravity is up
			SurvivePose lighthouse2objUp = {0}, object2objUp = {0};
			FLT accel_mag = norm3d(so->activations.accel);
			if (accel_mag != 0.0 && !isnan(accel_mag)) {
				quatfrom2vectors(object2objUp.Rot, so->activations.accel, up);
			} else {
				SV_WARN("Calibration didn't have valid IMU data for %s; couldn't establish 'up' vector.", so->codename);
				object2objUp.Rot[0] = 1.0;
			}

			// Calculate the pose of the lighthouse in this space
			ApplyPoseToPose(&lighthouse2objUp, &object2objUp, &lighthouse2obj);
			ApplyPoseToPose(&arb2world, &object2objUp, &arb2world);

			// Find what angle we need to rotate about Z by to get to 90 degrees.
			FLT ang = atan2(lighthouse2objUp.Pos[1], lighthouse2objUp.Pos[0]);
			FLT ang_target = M_PI / 2.;
			FLT euler[3] = {0, 0, ang_target - ang};
			SurvivePose objUp2World = {0};
			quatfromeuler(objUp2World.Rot, euler);

			ApplyPoseToPose(&arb2world, &objUp2World, &arb2world);
			ApplyPoseToPose(&obj2world, &arb2world, &object2arb);
			ApplyPoseToPose(&lighthouse2world, &arb2world, &lighthouse2arb);

			if (centerOnLh0) {
				sub3d(obj2world.Pos, obj2world.Pos, lighthouse2world.Pos)
```

**Multiple Base Stations:**

When calibrating with 2 base stations, the system:

1. First calibrates Base Station 0 (reference base station)
2. Then calibrates Base Station 1 using the established world coordinate system
3. Both base station poses are stored in the world coordinate system

The selection of reference base station is handled here:

```337:359:src/poser.c
		uint32_t lh_indices[NUM_GEN2_LIGHTHOUSES] = {0};
		uint32_t cnt = 0;

		uint32_t reference_basestation = survive_configi(so->ctx, "reference-basestation", SC_GET, 0);

		for (int lh = 0; lh < lighthouse_count; lh++) {
			SurvivePose lh2object = lighthouse_pose[lh];
			if (quatmagnitude(lh2object.Rot) != 0.0) {
				lh_indices[cnt] = lh;
				uint32_t lh0 = lh_indices[0];
				bool preferThisBSD = reference_basestation == 0
										 ? (so->ctx->bsd[lh].BaseStationID < so->ctx->bsd[lh0].BaseStationID)
										 : reference_basestation == so->ctx->bsd[lh].BaseStationID;
				if (preferThisBSD) {
					lh_indices[0] = lh;
					lh_indices[cnt] = lh0;
				}
				cnt++;
			}
		}

		struct SurviveContext *ctx = so->ctx;
		SV_INFO("Using LH %d (%08x) as reference lighthouse", lh_indices[0], so->ctx->bsd[lh_indices[0]].BaseStationID);
```

### Stage 5: Configuration Persistence

Once calibration is complete, the results are saved to a configuration file (typically `config.json` in `XDG_CONFIG_HOME/libsurvive`). This includes:

- Base station positions (pose: position + rotation quaternion)
- Base station calibration parameters (OOTX data)
- Base station IDs and modes
- Confidence values

**Saving Configuration:**

```443:476:src/survive_config.c
void config_set_lighthouse(config_group *lh_config, BaseStationData *bsd,
```

The configuration is automatically loaded on subsequent runs, allowing the system to skip calibration if the base stations haven't moved.

**Loading Configuration:**

```385:441:src/survive_config.c
bool config_read_lighthouse(config_group *lh_config, BaseStationData *bsd, uint8_t idx) {
	config_group *cg = lh_config + idx;
	uint8_t found = 0;
	for (int i = 0; i < NUM_GEN2_LIGHTHOUSES; i++) {
		uint32_t tmpIdx = 0xffffffff;
		cg = lh_config + idx;

		tmpIdx = config_read_uint32(cg, "index", 0xffffffff);

		if (tmpIdx == idx && i == idx) // assumes that lighthouses are stored in the config in order.
		{
			found = 1;
			break;
		}
	}

	//	assert(found); // throw an assertion if we didn't find it...  Is this good?  not necessarily?
	if (!found) {
		return false;
	}

	FLT defaults[7] = {0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};

	bsd->BaseStationID = config_read_uint32(cg, "id", 0);
	bsd->mode = config_read_uint32(cg, "mode", 0);

	config_read_float_array(cg, "pose", &bsd->Pose.Pos[0], defaults, 7);
	config_read_float_array(cg, "variance", &bsd->variance.Pos[0], defaults, 6);

	FLT accel[3] = {0};
	config_read_float_array(cg, "accel", accel, defaults, 3);
	for (int i = 0; i < 3; i++)
		bsd->accel[i] = accel[i];

	FLT cal[sizeof(bsd->fcal)] = { 0 };
	config_read_float_array(cg, "fcalphase", cal, defaults, 2);
	config_read_float_array(cg, "fcaltilt", cal + 2, defaults, 2);
	config_read_float_array(cg, "fcalcurve", cal + 4, defaults, 2);
	config_read_float_array(cg, "fcalgibpha", cal + 6, defaults, 2);
	config_read_float_array(cg, "fcalgibmag", cal + 8, defaults, 2);
	config_read_float_array(cg, "fcalogeephase", cal + 10, defaults, 2);
	config_read_float_array(cg, "fcalogeemag", cal + 12, defaults, 2);

	for (size_t i = 0; i < 2; i++) {
		bsd->fcal[i].phase = cal[0 + i];
		bsd->fcal[i].tilt = cal[2 + i];
		bsd->fcal[i].curve = cal[4 + i];
		bsd->fcal[i].gibpha = cal[6 + i];
		bsd->fcal[i].gibmag = cal[8 + i];
		bsd->fcal[i].ogeephase = cal[10 + i];
		bsd->fcal[i].ogeemag = cal[12 + i];
	}

	bsd->OOTXSet = config_read_uint32(cg, "OOTXSet", 0);
	bsd->PositionSet = config_read_uint32(cg, "PositionSet", 0);
	return true;
}
```

## Reprojection Model

The calibration parameters are used in the reprojection model to convert from 3D positions to angle measurements. For Base Station 2.0, this involves a complex non-linear model:

```62:103:src/survive_reproject_gen2.c
static inline FLT survive_reproject_axis_gen2(const BaseStationCal *bcal, FLT X, FLT Y, FLT Z, bool axis) {
	const FLT phase = bcal->phase;
	const FLT curve = bcal->curve;
	const FLT tilt = bcal->tilt;
	const FLT gibPhase = bcal->gibpha;
	const FLT gibMag = bcal->gibmag;
	const FLT ogeePhase = bcal->ogeephase;
	const FLT ogeeMag = bcal->ogeemag;

	FLT B = atan2(Z, X);

	FLT Ydeg = tilt + (axis ? -1 : 1) * LINMATHPI / 6.;
	FLT tanA = FLT_TAN(Ydeg);
	FLT normXZ = FLT_SQRT(X * X + Z * Z);

	FLT asinArg = tanA * Y / normXZ;
	FLT asinArg_sanitized = linmath_enforce_range(asinArg, -1, 1);

	FLT sinYdeg = FLT_SIN(Ydeg);
	FLT cosYdeg = FLT_COS(Ydeg);

	FLT sinPart = FLT_SIN(B - FLT_ASIN(asinArg_sanitized) + ogeePhase) * ogeeMag;

	FLT normXYZ = FLT_SQRT(X * X + Y * Y + Z * Z);

	FLT modAsinArg = linmath_enforce_range(Y / normXYZ / cosYdeg, -1, 1);

	FLT asinOut = FLT_ASIN(modAsinArg);

	FLT mod, acc;
	calc_cal_series(asinOut, &mod, &acc);

	FLT BcalCurved = sinPart + curve;
	FLT asinArg2 = linmath_enforce_range(asinArg + mod * BcalCurved / (cosYdeg - acc * BcalCurved * sinYdeg), -1, 1);

	FLT asinOut2 = FLT_ASIN(asinArg2);
	FLT sinOut2 = sin(B - asinOut2 + gibPhase);

	FLT rtn = B - asinOut2 + sinOut2 * gibMag - phase - LINMATHPI_2;
	assert(!isnan(rtn));
	return rtn;
}
```

This model accounts for various optical imperfections in the base station's laser system, including:
- Phase offsets
- Tilt corrections
- Curvature corrections
- Gibbous phase/magnitude corrections
- Ogee phase/magnitude corrections (Gen 2 specific)

## Calibration Requirements

For successful calibration:

1. **OOTX Data**: All base stations must broadcast their OOTX data. This typically takes 10-60 seconds per base station depending on signal quality. For setups with many base stations, this may take several minutes.

2. **Light Coverage**: 
   - **For 2 Base Stations**: The tracker must receive light from both base stations simultaneously or sequentially.
   - **For 3+ Base Stations**: The tracker should be placed where it can see as many base stations as possible. You may need to move the tracker to different positions to see all base stations.
   - The tracker should be placed in a location where it can "see" the base stations with clear line of sight.

3. **Stationary Period**: During calibration, the tracker(s) **MUST remain completely stationary**. The system uses stationary light measurements from multiple sensors to solve for base station positions. **No motion is required or recommended** - the calibration algorithm works best with stationary data. For multiple base stations, if not all are visible from one position, you may need to move the tracker to a new position and keep it stationary there until the remaining base stations calibrate.

4. **Sensor Visibility**: Multiple sensors on the tracker must be visible to each base station. The poser requires at least 4-5 sensor correspondences per base station to solve reliably. With multiple trackers, the system will use the best available tracker for each base station.

5. **Base Station Limit**: Do not exceed 16 base stations. The system will only recognize and calibrate the first 16 base stations detected. If you need more than 16 base stations, you may need to use multiple instances of libsurvive or modify the source code (changing `NUM_GEN2_LIGHTHOUSES` in `include/libsurvive/survive_types.h`).

## Calibration Timing

The calibration process typically proceeds as follows:

1. **0-10 seconds**: OOTX data collection begins. The system attempts to decode OOTX packets from both base stations.

2. **10-30 seconds**: Once OOTX data is collected, light data accumulation begins. The system collects angle measurements from multiple sensors.

3. **30-60 seconds**: When sufficient data is available, the poser runs and solves for base station positions. This may happen multiple times as more data becomes available.

4. **60+ seconds**: Calibration stabilizes. The system continues to refine positions but the major calibration is complete.

## Troubleshooting

**Base stations not calibrating:**

- Ensure both base stations are powered on and in tracking mode
- Check that the tracker can see both base stations (no obstructions)
- Verify OOTX data is being received (check logs with `--v 100`)
- Try moving the tracker to a different location with better visibility

**Calibration takes too long:**

- Ensure good line-of-sight between tracker and base stations
- Check for reflections or interference
- Verify base stations are properly synchronized (should be automatic for Gen 2)

**Inaccurate tracking after calibration:**

- Force recalibration with `--force-calibrate`
- Verify base stations haven't moved since last calibration
- Check that OOTX data was successfully collected (in config file)

## Summary

The calibration process in libsurvive is a sophisticated multi-stage process that:

1. Collects factory calibration data (OOTX) from base stations
2. Processes light activation data from trackers
3. Solves for relative positions using geometric algorithms
4. Establishes a consistent world coordinate system
5. Persists results for future use

For a setup with 2 Base Station 2.0 and 1 Tracker 3.0, the system will calibrate both base stations relative to the tracker, establish a world coordinate system based on gravity and the reference base station, and save this information for future tracking sessions.

