<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meridian Flip Timeline Simulator</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom styles for the timeline */
        .timeline-segment {
            transition: all 0.3s ease-out;
            border-radius: 4px;
            box-shadow: 0 1px 3px rgba(0, 0, 0, 0.4); /* Darker shadow for contrast */
            min-width: 3px; /* Ensure visibility */
            cursor: pointer;
        }
        .timeline-segment:hover .tooltip {
            visibility: visible;
            opacity: 1;
        }
        .tooltip {
            visibility: hidden;
            opacity: 0;
            transition: opacity 0.3s;
            z-index: 50;
            /* Dark theme tooltip background */
            background-color: #1f2937; 
            border: 1px solid #4b5563;
        }
        /* Meridian Line (t=0) - Amber */
        .meridian-line {
            width: 4px;
            height: 100%;
            background-color: #f59e0b; /* Amber 500 */
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            z-index: 10;
        }
        .meridian-line::after {
            content: 'MERIDIAN (t=0)';
            position: absolute;
            top: 0;
            left: 50%;
            transform: translate(-50%, -120%);
            font-size: 0.75rem;
            font-weight: 600;
            color: #f59e0b;
            text-align: center;
            width: 100px;
        }
        /* Timeline Container */
        .timeline-container {
            position: relative;
            height: 300px; 
            display: flex;
            align-items: center;
            overflow-x: auto; 
            border: 2px solid #374151; /* Gray 700 border */
            border-radius: 8px;
            padding: 1rem 0;
            background-color: #1f2937; /* Gray 800 background */
        }
        /* Input Field Styling for Dark Theme */
        input[type="number"], input[type="range"] {
            background-color: #374151; /* Gray 700 */
            border-color: #4b5563; /* Gray 600 */
            color: #f3f4f6; /* Light text */
        }
        input[type="range"]::-webkit-slider-thumb {
            background: #60a5fa; /* Blue 400 */
        }
        input[type="range"]::-moz-range-thumb {
            background: #60a5fa; /* Blue 400 */
        }

        /* Time Axis Styles (New Feature) */
        .time-axis-container {
            position: relative;
            width: 100%;
            height: 80px; /* Increased height for labels */
            margin-top: 1rem;
        }
        .time-axis-line {
            position: absolute;
            width: 100%;
            height: 2px;
            background-color: #f87171; /* Red 400 */
            top: 50%;
            transform: translateY(-50%);
        }
        .time-marker {
            position: absolute;
            width: 2px;
            height: 15px;
            background-color: #f87171;
            top: 50%;
            transform: translate(-50%, -50%);
            z-index: 20;
        }
        .time-marker-label {
            position: absolute;
            bottom: 100%;
            left: 50%;
            transform: translateX(-50%) translateY(5px);
            font-size: 0.75rem;
            text-align: center;
            width: 100px;
        }
        .meridian-axis-marker {
            background-color: #f59e0b;
            height: 30px;
            width: 3px;
            top: 50%;
            transform: translate(-50%, -50%);
        }
        .meridian-axis-marker .time-marker-label {
            color: #f59e0b;
            font-weight: bold;
        }
    </style>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                        mono: ['Consolas', 'Menlo', 'monospace'],
                    },
                    colors: {
                        'primary-green': '#10b981', /* Emerald 500 */
                        'downtime-red': '#f87171', /* Red 400 */
                        'wait-orange': '#fb923c', /* Orange 400 */
                        'slew-blue': '#60a5fa', /* Blue 400 */
                        'card-bg': '#1f2937', /* Gray 800 */
                        'subcard-bg': '#111827', /* Gray 900 */
                    }
                }
            }
        }
    </script>
</head>
<body class="bg-gray-900 p-4 md:p-8 font-sans text-gray-100">

    <div class="max-w-4xl mx-auto bg-gray-800 shadow-2xl rounded-xl p-6 md:p-8">
        <h1 class="text-3xl font-bold mb-4 text-gray-50 border-b border-gray-700 pb-2">N.I.N.A. Meridian Flip Simulator</h1>

        <!-- Configuration Inputs - All in one row -->
        <div class="grid grid-cols-2 md:grid-cols-4 gap-6 mb-8">
            
            <!-- T1 Input (N.I.N.A.) -->
            <div class="bg-gray-700 p-4 rounded-lg shadow-xl">
                <label for="t1" class="block text-sm font-medium text-gray-300">Minutes after meridian (T1)</label>
                <input type="number" id="t1" value="0" min="0" class="mt-1 block w-full border border-gray-600 rounded-md shadow-sm p-2 font-mono text-center" oninput="calculateFlip()">
                <p class="text-xs text-gray-400 mt-1">Flip earliest start time (min)</p>
            </div>

            <!-- T2 Input (N.I.N.A.) -->
            <div class="bg-gray-700 p-4 rounded-lg shadow-xl">
                <label for="t2" class="block text-sm font-medium text-gray-300">Max. minutes after meridian (T2)</label>
                <input type="number" id="t2" value="15" min="0" class="mt-1 block w-full border border-gray-600 rounded-md shadow-sm p-2 font-mono text-center" oninput="calculateFlip()">
                <p class="text-xs text-gray-400 mt-1">Flip software deadline (min)</p>
            </div>

            <!-- T_Pause Input (N.I.N.A.) -->
            <div class="bg-gray-700 p-4 rounded-lg shadow-xl">
                <label for="t_pause" class="block text-sm font-medium text-gray-300">Pause before meridian</label>
                <input type="number" id="t_pause" value="0" min="0" class="mt-1 block w-full border border-gray-600 rounded-md shadow-sm p-2 font-mono text-center" oninput="calculateFlip()">
                <p class="text-xs text-gray-400 mt-1">Stops tracking before Meridian (min)</p>
            </div>
            
            <!-- Mount Limit Input (Physical Constraint) -->
            <div class="bg-yellow-900/50 p-4 rounded-lg shadow-xl border border-yellow-700">
                <label for="t_mount_limit" class="block text-sm font-medium text-yellow-300">Max. Mount Tracking Past Meridian</label>
                <input type="number" id="t_mount_limit" value="20" min="0" class="mt-1 block w-full border border-yellow-800 rounded-md shadow-sm p-2 font-mono text-center" oninput="calculateFlip()">
                <p class="text-xs text-yellow-400 mt-1">Absolute physical/driver limit (min)</p>
            </div>

        </div>

        <!-- Timeline Visualization (Moved to the top) -->
        <div class="mb-8">
            <h2 class="text-xl font-semibold mb-4 text-gray-200">Timeline Visualization (Total Downtime: <span id="total-time-min" class="font-mono text-downtime-red font-bold"></span>)</h2>
            <div id="timeline-chart" class="timeline-container">
                <div class="meridian-line"></div>
                <!-- Timeline segments will be injected here -->
            </div>
            <p class="text-xs text-gray-400 mt-3 text-center">Duration-based scale. Hover over segments for detail. (Scroll horizontally if needed)</p>
            
            <!-- NEW: Time Axis Visualization (Coordinate-based, Non-scaled) -->
            <div class="mt-6">
                <h3 class="text-lg font-semibold mb-2 text-gray-300">Key Timing Events (Relative to Meridian, $t=0$)</h3>
                <div id="time-axis-chart" class="time-axis-container bg-gray-700/50 rounded-lg p-2">
                    <div class="time-axis-line"></div>
                </div>
            </div>
        </div>


        <!-- Dynamic Settings & Scenario Selector -->
        <div class="bg-blue-900/40 p-6 rounded-xl mb-8 border border-blue-800">
            <h2 class="text-xl font-semibold mb-4 text-blue-300">Scenario Selector: Last Exposure End Time</h2>
            
            <!-- Exposure End Time Slider -->
            <div class="mb-6">
                <label for="t_end_exp" class="block text-lg font-medium text-gray-200 mb-2">
                    Simulated End Time of Last Exposure: 
                    <span id="t_end_exp_display" class="font-mono text-blue-400 ml-2"></span>
                </label>
                <input type="range" id="t_end_exp" value="0" min="-180" max="300" step="10" 
                       class="w-full h-2 bg-blue-800 rounded-lg appearance-none cursor-pointer" oninput="calculateFlip()">
                <div class="flex justify-between text-xs text-gray-400 mt-1">
                    <span>-3m (Well Before Meridian)</span>
                    <span class="text-gray-200 font-semibold">Meridian (0s)</span>
                    <span>+5m (Well After Meridian)</span>
                </div>
            </div>

            <!-- System Constants -->
            <div class="bg-blue-900 p-3 rounded-lg text-xs text-blue-400">
                <h3 class="font-bold mb-1">Post-Flip Time Assumptions:</h3>
                <ul class="list-disc list-inside space-y-0.5">
                    <li>Slew Time: 45s (Fixed downtime for mount flip)</li>
                </ul>
            </div>
        </div>


        <!-- Results and Summary -->
        <div class="mb-8 p-6 bg-green-900/40 border-2 border-green-700 rounded-xl">
            <h2 class="text-2xl font-bold mb-4 text-green-300">Flip Outcome Summary</h2>
            <div id="summary" class="text-base text-gray-200 space-y-2">
                <!-- Summary will be injected here -->
            </div>
        </div>

    </div>

    <script>
        // System Constants (Used for Post-Flip calculation)
        const SYSTEM_CONSTANTS = {
            T_Slew: 45,       // Mount Slew/Flip Time (seconds)
        };

        /**
         * Converts a time in seconds to a formatted string (e.g., 5m 30s).
         * @param {number} totalSeconds
         * @returns {string}
         */
        function formatTime(totalSeconds) {
            const sign = totalSeconds < 0 ? "-" : "";
            const absSeconds = Math.abs(totalSeconds);
            const minutes = Math.floor(absSeconds / 60);
            const seconds = absSeconds % 60;

            const secPart = seconds.toFixed(0);

            if (minutes > 0) {
                return `${sign}${minutes}m ${secPart.padStart(2, '0')}s`;
            }
            return `${sign}${secPart}s`;
        }
        
        /**
         * Renders the simple non-scaled time axis below the main timeline.
         */
        function renderTimeAxis(t_DowntimeStart, t_FlipStart, T_Effective_Deadline) {
            const axisChart = document.getElementById('time-axis-chart');
            axisChart.innerHTML = '<div class="time-axis-line"></div>';

            const containerWidth = axisChart.clientWidth || 800;
            const leftPadding = 0.05 * containerWidth; // 5% padding
            const rightPadding = 0.05 * containerWidth;
            const contentWidth = containerWidth - leftPadding - rightPadding;

            // Determine the max absolute time value to set the scale (e.g., if T_Effective_Deadline is 30m)
            const maxTime = Math.max(
                Math.abs(t_DowntimeStart), 
                Math.abs(t_FlipStart), 
                T_Effective_Deadline
            ) + 60; // Add 60s buffer
            
            const timeSpan = maxTime * 2; // Time span from -maxTime to +maxTime
            const scaleFactor = contentWidth / timeSpan;

            // Position of the meridian (t=0)
            const meridianPos = (maxTime * scaleFactor) + leftPadding;

            const timeMarkers = [
                { time: t_DowntimeStart, label: 'Tracking Stops', color: 'text-red-400', special: 'stop' },
                { time: t_FlipStart, label: 'Flip Starts (T1)', color: 'text-blue-400', special: 'flip' },
                { time: T_Effective_Deadline, label: 'Mount Deadline', color: 'text-yellow-400', special: 'limit' },
            ];

            // 1. Render Meridian Marker
            const meridianMarker = document.createElement('div');
            meridianMarker.className = 'time-marker meridian-axis-marker';
            meridianMarker.style.left = `${meridianPos}px`;
            meridianMarker.innerHTML = `<span class="time-marker-label">Meridian (${formatTime(0)})</span>`;
            axisChart.appendChild(meridianMarker);
            
            // 2. Render Key Time Markers
            timeMarkers.forEach(marker => {
                const markerTime = marker.time;
                const position = ((markerTime + maxTime) * scaleFactor) + leftPadding;
                
                // Only render if within the visual range
                if (position > leftPadding && position < containerWidth - rightPadding) {
                    const markerElement = document.createElement('div');
                    markerElement.className = `time-marker ${marker.color}`;
                    markerElement.style.left = `${position}px`;
                    
                    const markerLabel = document.createElement('span');
                    markerLabel.className = `time-marker-label ${marker.color} -mt-5`;
                    markerLabel.innerHTML = `${marker.label}<br/>(${formatTime(markerTime)})`;
                    
                    markerElement.appendChild(markerLabel);
                    axisChart.appendChild(markerElement);
                }
            });
        }


        /**
         * Converts an array of timeline segments into HTML for visualization.
         */
        function renderTimeline(segments, minTime, totalTimeSpan) {
            const timelineChart = document.getElementById('timeline-chart');
            // Remove Mount Limit Line logic from here
            timelineChart.innerHTML = '<div class="meridian-line"></div>'; 
            
            const containerWidth = timelineChart.clientWidth || 800;
            const contentWidth = containerWidth * 0.95; // Use 95% for padding
            const scaleFactor = contentWidth / totalTimeSpan; 

            // Calculate the pixel position of t=0 (Meridian) relative to the start of the visible span (minTime)
            const meridianOffsetFromStart = Math.abs(minTime) * scaleFactor;
            const leftPadding = (containerWidth * 0.05) / 2;
            const meridianOffsetPixels = meridianOffsetFromStart + leftPadding;

            // 1. Render the segments
            segments.forEach((segment) => {
                const duration = segment.duration;
                const width = Math.max(3, duration * scaleFactor); // Min width of 3px
                
                const segmentElement = document.createElement('div');
                segmentElement.className = `timeline-segment ${segment.color} h-12 relative flex items-center justify-center text-white text-xs whitespace-nowrap px-1 font-semibold`;
                
                // Calculate position based on the start time (relative to minTime)
                const positionFromStartOfSpan = segment.time_start - minTime;
                // Add the leftPadding to correctly align with the Meridian line
                const positionOffset = positionFromStartOfSpan * scaleFactor + leftPadding;

                // Adjust the main position for the segments
                segmentElement.style.position = 'absolute';
                segmentElement.style.left = `${positionOffset}px`;
                segmentElement.style.width = `${width}px`;
                segmentElement.style.zIndex = '20'; // Above the meridian line

                const timeStartLabel = segment.time_start < 0 ? `${formatTime(segment.time_start)} (Before Meridian)` : `${formatTime(segment.time_start)} (After Meridian)`;
                const timeEndLabel = segment.time_end < 0 ? `${formatTime(segment.time_end)} (Before Meridian)` : `${formatTime(segment.time_end)} (After Meridian)`;

                // Tooltip positioned above the segment
                segmentElement.innerHTML = `
                    <span class="text-shadow-sm pointer-events-none">${segment.short_name}</span>
                    <div class="tooltip absolute bottom-full mb-3 p-3 bg-gray-800 text-white rounded-md shadow-2xl text-center text-sm">
                        <p class="font-bold text-lg mb-1">${segment.name}</p>
                        <p>Duration: <span class="font-mono">${formatTime(duration)}</span></p>
                        <p>Starts at: <span class="font-mono">${timeStartLabel}</span></p>
                        <p>Ends at: <span class="font-mono">${timeEndLabel}</span></p>
                    </div>
                `;
                
                timelineChart.appendChild(segmentElement);
            });
            
            // Re-center the meridian line
            const meridianLine = document.querySelector('.meridian-line');
            meridianLine.style.left = `${meridianOffsetPixels}px`;
            
            // Scroll to center the meridian line on load/re-calc
            timelineChart.scrollLeft = meridianOffsetPixels - (timelineChart.clientWidth / 2);

        }


        /**
         * Calculates the total downtime and generates the timeline data points.
         */
        function calculateFlip() {
            // 1. Get User Inputs (converted to seconds where necessary)
            const T1_min = parseFloat(document.getElementById('t1').value) || 0;
            const T2_min = parseFloat(document.getElementById('t2').value) || 0;
            const T_Pause_min = parseFloat(document.getElementById('t_pause').value) || 0;
            const T_End_Exp = parseFloat(document.getElementById('t_end_exp').value) || 0; // Scenario slider value
            const T_MountLimit_min = parseFloat(document.getElementById('t_mount_limit').value) || 0; 

            // Update slider display
            document.getElementById('t_end_exp_display').textContent = formatTime(T_End_Exp);

            const T1_sec = T1_min * 60;
            const T2_sec_software = T2_min * 60; // T2 as set in NINA
            const T_MountLimit_sec = T_MountLimit_min * 60; // Mount's hard limit
            const T_Pause_sec = T_Pause_min * 60;

            // The effective absolute deadline is the lesser of the software T2 and the mount's physical limit.
            const T_Effective_Deadline = Math.min(T2_sec_software, T_MountLimit_sec);

            let T_Wait = 0;
            let t_DowntimeStart = 0;
            let t_FlipStart = 0;
            let scenarioDescription = "";
            let t_StopTracking = 0;
            
            // --- 2. Determine Tracking Stop Time (t_StopTracking) ---
            if (T_Pause_min > 0) {
                // Scenario 1: Forced Pause is set. Tracking stops at -T_Pause_sec.
                t_StopTracking = -T_Pause_sec;
                scenarioDescription = "Forced Pause: Tracking stopped early to avoid an obstruction.";
            } else if (T_End_Exp > T_Effective_Deadline) {
                // Scenario 2: Exposure ends AFTER the deadline. NINA would abort/skip the last sub and flip at the deadline.
                t_StopTracking = T_Effective_Deadline - 1; 
                t_FlipStart = T_Effective_Deadline;
                T_Wait = 1; // Minimal wait time to force the flip at the deadline
                scenarioDescription = "Deadline Reached: Last exposure aborted/skipped. Flip forced at deadline.";
            } else {
                // Scenario 3: Normal case. Tracking stops when the last successful exposure finishes.
                t_StopTracking = T_End_Exp;
                scenarioDescription = `Tracking Stopped: Last exposure ended at ${formatTime(T_End_Exp)}.`;
            }
            
            // --- 3. Determine Flip Start Time (t_FlipStart) and Wait Time (T_Wait) ---
            t_DowntimeStart = t_StopTracking;

            if (scenarioDescription.includes("Deadline Reached")) {
                // Handled in Scenario 2 above. T_FlipStart is T_Effective_Deadline.
            } else if (t_DowntimeStart < T1_sec) {
                // Flip must wait until the earliest time (T1)
                t_FlipStart = T1_sec;
                T_Wait = t_FlipStart - t_DowntimeStart;
            } else {
                // Flip starts immediately after tracking stops (since t_DowntimeStart is in the T1/T_Effective_Deadline window)
                t_FlipStart = t_DowntimeStart;
                T_Wait = 0;
            }

            
            // --- 4. Calculate Post-Flip Recovery Time (T_PostFlip) ---
            // Simplified Post-Flip: Only Slew Time (45s)
            const T_PostFlip = SYSTEM_CONSTANTS.T_Slew;
            let current_time = t_FlipStart;
            const segments = [];

            // --- Timeline Data Structure ---
            
            // 0. Initial Tracking (Pre-Downtime)
            const T_Tracking_Context = 120; // Show 2 minutes of tracking context
            segments.push({ 
                name: 'IMAGING (CONTEXT)', 
                short_name: 'IMAGING', 
                duration: T_Tracking_Context, 
                color: 'bg-primary-green/70', 
                time_start: t_DowntimeStart - T_Tracking_Context,
                time_end: t_DowntimeStart
            });

            // 1. Pre-Flip Wait (T_Wait)
            if (T_Wait > 0) {
                segments.push({ 
                    name: 'IDLE WAIT', 
                    short_name: 'WAIT', 
                    duration: T_Wait, 
                    color: T_Pause_min > 0 ? 'bg-downtime-red' : 'bg-wait-orange', 
                    time_start: t_DowntimeStart,
                    time_end: t_FlipStart
                });
            }
            
            // --- Post-Flip Sequence ---

            // 2. Flip Execution (Slew)
            const t_slew_end = current_time + SYSTEM_CONSTANTS.T_Slew;
            segments.push({
                name: 'SLEW/FLIP', 
                short_name: 'FLIP', 
                duration: SYSTEM_CONSTANTS.T_Slew, 
                color: 'bg-slew-blue', 
                time_start: current_time,
                time_end: t_slew_end
            });
            current_time = t_slew_end;

            // 3. Resuming Imaging
            segments.push({
                name: 'IMAGING RESUMES', 
                short_name: 'RESUME', 
                duration: 60, // Arbitrary 60s for visual continuity
                color: 'bg-primary-green/90', 
                time_start: current_time,
                time_end: current_time + 60
            });


            // --- 5. Final Calculations and Summary Output ---
            const T_Downtime = T_Wait + T_PostFlip;
            const time_meridian_to_resume = current_time; // This is the total time from t=0 to imaging resume

            // Determine the visualization span
            const minTime = segments.reduce((min, s) => Math.min(min, s.time_start), 0);
            const maxTime = segments.reduce((max, s) => Math.max(max, s.time_end), 0); 
            const totalTimeSpan = maxTime - minTime;


            let summaryHTML = `
                <p><strong>Scenario Status:</strong> <span class="text-blue-400">${scenarioDescription}</span></p>
                <p><strong>Effective Flip Deadline:</strong> <span class="font-mono text-base text-yellow-500">${formatTime(T_Effective_Deadline)}</span> (Minimum of T2: ${formatTime(T2_sec_software)} and Mount Limit: ${formatTime(T_MountLimit_sec)})</p>
                
                <hr class="my-3 border-gray-700">

                <p><strong>Tracking Stopped At:</strong> <span class="font-mono text-lg text-red-400">${formatTime(t_DowntimeStart)}</span></p>
                <p><strong>Flip Initiated At:</strong> <span class="font-mono text-lg text-blue-400">${formatTime(t_FlipStart)}</span></p>
                
                <hr class="my-3 border-gray-700">

                <p><strong>Pre-Flip Wait Time (T_Wait):</strong> <span class="font-mono text-wait-orange text-lg">${formatTime(T_Wait)}</span></p>
                <p><strong>Post-Flip Recovery (T_PostFlip):</strong> <span class="font-mono text-slew-blue text-lg">${formatTime(T_PostFlip)}</span> (Slew Time Only)</p>
                
                <hr class="my-3 border-gray-700">

                <p class="mt-4 text-xl"><strong>TOTAL DOWNTIME:</strong> <span class="text-downtime-red font-bold font-mono">${formatTime(T_Downtime)}</span></p>
                <p class="mt-1 text-sm text-gray-400">Total time from Meridian (t=0) to Imaging Resumption: <span class="font-mono">${formatTime(time_meridian_to_resume)}</span></p>
            `;
            document.getElementById('summary').innerHTML = summaryHTML;
            document.getElementById('total-time-min').textContent = formatTime(T_Downtime);
            
            // --- 6. Render Visualization ---
            renderTimeline(segments, minTime, totalTimeSpan);
            renderTimeAxis(t_DowntimeStart, t_FlipStart, T_Effective_Deadline);
        }

        // Initialize the calculation on page load
        window.onload = () => {
            // Set the slider's initial value to 0s to center it on meridian
            document.getElementById('t_end_exp').value = 0; 

            calculateFlip();
            // Recalculate on resize to maintain responsiveness
            window.addEventListener('resize', calculateFlip); 
        };
    </script>
</body>
</html>
