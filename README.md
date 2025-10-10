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
            overflow-x: auto; /* Allow horizontal scrolling only */
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
            height: 100px; /* Increased height to accommodate bottom labels */
            margin-top: 1rem;
            overflow-x: hidden; /* Hide default scrollbar, JS will manage pan/zoom */
            cursor: grab;
        }
        .time-axis-container:active {
            cursor: grabbing;
        }

        .time-axis-line-wrapper {
            position: absolute;
            height: 100%;
            left: 0;
            top: 0;
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
        /* Marker Label Placements */
        .time-marker-label {
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            font-size: 0.75rem;
            text-align: center;
            width: 100px;
        }
        .label-top {
            bottom: 100%;
            transform: translateX(-50%) translateY(5px);
        }
        .label-bottom {
            top: 100%;
            transform: translateX(-50%) translateY(-5px);
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
            bottom: 100%; /* Keep meridian label on top */
            transform: translateX(-50%) translateY(5px);
        }
    </style>
</head>
<body class="bg-gray-900 text-gray-100 p-6 min-h-screen font-sans">

    <div class="max-w-4xl mx-auto">
        <h1 class="text-3xl font-extrabold mb-8 text-blue-400">N.I.N.A. Meridian Flip Simulator</h1>

        <!-- Input Parameters -->
        <div class="grid grid-cols-1 md:grid-cols-4 gap-4 mb-8">
            <!-- T1 Input -->
            <div class="bg-gray-800 p-4 rounded-lg shadow-xl border border-gray-700">
                <label for="t1" class="block text-sm font-medium text-gray-400 mb-1">Minutes after meridian (T1)</label>
                <input type="number" id="t1" value="0" min="0" oninput="calculateFlip()"
                       class="w-full p-2 border rounded-md focus:ring-blue-500 focus:border-blue-500 text-lg font-mono">
                <p class="text-xs text-gray-500 mt-1">Flip earliest start time (min)</p>
            </div>

            <!-- T2 Input -->
            <div class="bg-gray-800 p-4 rounded-lg shadow-xl border border-gray-700">
                <label for="t2" class="block text-sm font-medium text-gray-400 mb-1">Max. minutes after meridian (T2)</label>
                <input type="number" id="t2" value="15" min="0" oninput="calculateFlip()"
                       class="w-full p-2 border rounded-md focus:ring-blue-500 focus:border-blue-500 text-lg font-mono">
                <p class="text-xs text-gray-500 mt-1">Flip software deadline (min)</p>
            </div>

            <!-- Pause Before Meridian Input -->
            <div class="bg-gray-800 p-4 rounded-lg shadow-xl border border-gray-700">
                <label for="t_pause" class="block text-sm font-medium text-gray-400 mb-1">Pause before meridian</label>
                <input type="number" id="t_pause" value="0" min="0" oninput="calculateFlip()"
                       class="w-full p-2 border rounded-md focus:ring-blue-500 focus:border-blue-500 text-lg font-mono">
                <p class="text-xs text-gray-500 mt-1">Stops tracking before Meridian (min)</p>
            </div>

            <!-- Mount Tracking Limit Input -->
            <div class="bg-gray-800 p-4 rounded-lg shadow-xl border border-gray-700">
                <label for="t_mount_limit" class="block text-sm font-medium text-gray-400 mb-1">Max. Mount Tracking Past Meridian</label>
                <input type="number" id="t_mount_limit" value="20" min="0" oninput="calculateFlip()"
                       class="w-full p-2 border rounded-md focus:ring-blue-500 focus:border-blue-500 text-lg font-mono">
                <p class="text-xs text-gray-500 mt-1">Absolute physical/driver limit (min)</p>
            </div>
        </div>

        <!-- Timeline Visualization -->
        <h2 class="text-xl font-bold mb-3 text-gray-200">Timeline Visualization (<span class="text-downtime-red">Total Downtime: <span id="total-time-min">0s</span></span>)</h2>
        <div id="timeline-chart" class="timeline-container w-full">
            <!-- Segments and Meridian Line are rendered here by JavaScript -->
        </div>
        <p class="text-xs text-gray-500 text-center mt-2">Timeline is scaled dynamically. Hover over segments for detail. (Scroll horizontally if needed)</p>

        <!-- Key Timing Events (Zoomable Axis) -->
        <h2 class="text-xl font-bold mb-3 mt-8 text-gray-200">Key Timing Events (Zoomable Reference Axis)</h2>
        <div id="time-axis-chart" class="time-axis-container">
            <div id="time-axis-wrapper" class="time-axis-line-wrapper">
                <!-- Time axis line and markers are rendered here by JavaScript -->
            </div>
        </div>
        <p class="text-xs text-gray-500 text-center mt-2">Use scroll wheel to **zoom** and click-and-drag to **pan** this axis.</p>


        <!-- Scenario Selector -->
        <div class="bg-gray-800 p-6 rounded-lg shadow-xl border border-gray-700 mt-8">
            <h2 class="text-xl font-bold mb-4 text-blue-300">Scenario Selector: Last Exposure End Time</h2>
            
            <div class="flex justify-between items-center mb-4">
                <label for="t_end_exp" class="block text-sm font-medium text-gray-300">Simulated End Time of Last Exposure:</label>
                <span id="t_end_exp_display" class="font-mono text-lg text-yellow-400">0s</span>
            </div>
            
            <input type="range" id="t_end_exp" min="-300" max="300" step="1" value="0" oninput="calculateFlip()"
                   class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer range-lg">
            
            <div class="flex justify-between text-xs text-gray-500 mt-1">
                <span>-5m (Well Before Meridian)</span>
                <span class="text-amber-400 font-semibold">Meridian (0s)</span>
                <span>+5m (Well After Meridian)</span>
            </div>
            
            <h3 class="text-base font-semibold mt-4 text-gray-400">Post-Flip Time Assumptions:</h3>
            <ul class="text-sm list-disc list-inside ml-4 text-gray-500">
                <li>Slew Time: 45s (Fixed downtime for mount flip)</li>
            </ul>
        </div>
        
        <!-- Flip Outcome Summary -->
        <div class="bg-green-900/30 p-6 rounded-lg shadow-xl border border-green-700 mt-8">
            <h2 class="text-xl font-bold mb-4 text-green-400">Flip Outcome Summary</h2>
            <div id="summary" class="text-gray-200 space-y-1">
                <!-- Summary details rendered here by JavaScript -->
            </div>
        </div>
    </div>

    <script>
        // State variables for Pan and Zoom on the Time Axis
        let axisCurrentScale = 1.0;
        let isDragging = false;
        let startX;
        let startScrollLeft;

        // System Constants (Used for Post-Flip calculation)
        const SYSTEM_CONSTANTS = {
            T_Slew: 45,       // Mount Slew/Flip Time (seconds)
        };
        
        // Fixed pixel width for the main visualization bar chart
        const FIXED_TIMELINE_WIDTH_PX = 2000; 

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
         * Renders the zoomable/pannable time axis below the main timeline.
         */
        function renderTimeAxis(t_DowntimeStart, t_FlipStart, T_Effective_Deadline) {
            const axisChart = document.getElementById('time-axis-chart');
            const axisWrapper = document.getElementById('time-axis-wrapper');
            axisWrapper.innerHTML = '<div class="time-axis-line"></div>';

            const containerWidth = axisChart.clientWidth || 800;
            const contentWidthBase = containerWidth * 0.95; 

            // --- Determine Scale and Span for Zoom ---
            // Max time determines the visual range when scale is 1.0
            const maxTime = Math.max(
                Math.abs(t_DowntimeStart), 
                Math.abs(t_FlipStart), 
                T_Effective_Deadline,
                300 // Minimum span of 5 minutes (300s) on either side
            ); 
            
            const timeSpan = maxTime * 2; // Time span from -maxTime to +maxTime
            const baseScaleFactor = contentWidthBase / timeSpan;
            
            // Apply current zoom scale
            const currentScaleFactor = baseScaleFactor * axisCurrentScale;
            const scaledContentWidth = timeSpan * currentScaleFactor;
            
            // Set the wrapper width
            axisWrapper.style.width = `${scaledContentWidth}px`;
            
            // --- Calculate Positions ---
            const contentStart = (maxTime * currentScaleFactor);

            // Position of the meridian (t=0)
            const meridianPos = contentStart; 

            const timeMarkers = [
                { time: t_DowntimeStart, label: 'Tracking Stops', color: 'text-red-400', special: 'stop' },
                { time: t_FlipStart, label: 'Flip Starts (T1)', color: 'text-blue-400', special: 'flip' },
                { time: T_Effective_Deadline, label: 'Mount Deadline', color: 'text-yellow-400', special: 'limit' },
            ];

            // Use an index to alternate placement (0 = top, 1 = bottom)
            let placementIndex = 0; 
            
            // 1. Render Meridian Marker
            const meridianMarker = document.createElement('div');
            meridianMarker.className = 'time-marker meridian-axis-marker';
            meridianMarker.style.left = `${meridianPos}px`;
            meridianMarker.innerHTML = `<span class="time-marker-label label-top">Meridian (${formatTime(0)})</span>`;
            axisWrapper.appendChild(meridianMarker);
            
            // 2. Render Key Time Markers
            timeMarkers.forEach(marker => {
                const markerTime = marker.time;
                // Position relative to the *start* of the scaled content (which is -maxTime)
                const position = (markerTime + maxTime) * currentScaleFactor; 
                
                // Skip rendering the marker if it's the Meridian time, to prevent overlap (handled by special marker)
                if (markerTime === 0 && marker.label === 'Flip Starts (T1)') {
                    // Check if Flip Starts and Meridian are the same time
                    return; 
                }

                const markerElement = document.createElement('div');
                // Use marker.color class for the label color
                markerElement.className = `time-marker ${marker.color.replace('text-', 'bg-')}`;
                markerElement.style.left = `${position}px`;
                
                const markerLabel = document.createElement('span');
                
                // Alternate placement class
                const placementClass = placementIndex % 2 === 0 ? 'label-top' : 'label-bottom';
                
                markerLabel.className = `time-marker-label ${marker.color} ${placementClass}`;
                markerLabel.innerHTML = `${marker.label}<br/>(${formatTime(markerTime)})`;
                
                markerElement.appendChild(markerLabel);
                axisWrapper.appendChild(markerElement);

                // Increment index only if we rendered the marker (to handle T1=0 case)
                placementIndex++;
            });

            // Center the view on the Meridian initially or after zoom
            const scrollCenterPosition = meridianPos - (containerWidth / 2);
            axisChart.scrollLeft = scrollCenterPosition;
        }


        /**
         * Renders the main non-zoomable, scrollable bar chart visualization.
         */
        function renderTimeline(segments, minTime, totalTimeSpan) {
            const timelineChart = document.getElementById('timeline-chart');
            timelineChart.innerHTML = '<div class="meridian-line"></div>'; 
            
            const containerWidth = timelineChart.clientWidth || 800;
            
            // The scale factor is fixed based on the total time span (in seconds) 
            // and the fixed pixel width defined globally.
            const scaleFactor = FIXED_TIMELINE_WIDTH_PX / totalTimeSpan;
            const scaledContentWidth = FIXED_TIMELINE_WIDTH_PX;
            
            const innerWrapper = document.createElement('div');
            innerWrapper.style.position = 'absolute';
            innerWrapper.style.height = '100%';
            innerWrapper.style.width = `${scaledContentWidth + 100}px`; // Add buffer
            innerWrapper.style.left = '0';
            innerWrapper.style.top = '0';

            const leftPadding = 50; // Fixed padding for visual comfort
            const meridianOffsetFromStart = Math.abs(minTime) * scaleFactor;
            const meridianOffsetPixels = meridianOffsetFromStart + leftPadding;
            
            // 1. Render the segments onto the inner wrapper
            segments.forEach((segment) => {
                const duration = segment.duration;
                const width = Math.max(3, duration * scaleFactor); 
                
                const segmentElement = document.createElement('div');
                segmentElement.className = `timeline-segment ${segment.color} h-12 absolute flex items-center justify-center text-white text-xs whitespace-nowrap px-1 font-semibold`;
                
                const positionFromStartOfSpan = segment.time_start - minTime;
                const positionOffset = positionFromStartOfSpan * scaleFactor + leftPadding;

                segmentElement.style.left = `${positionOffset}px`;
                segmentElement.style.width = `${width}px`;
                segmentElement.style.zIndex = '20'; 

                const timeStartLabel = segment.time_start < 0 ? `${formatTime(segment.time_start)} (Before Meridian)` : `${formatTime(segment.time_start)} (After Meridian)`;
                const timeEndLabel = segment.time_end < 0 ? `${formatTime(segment.time_end)} (Before Meridian)` : `${formatTime(segment.time_end)} (After Meridian)`;

                segmentElement.innerHTML = `
                    <span class="text-shadow-sm pointer-events-none">${segment.short_name}</span>
                    <div class="tooltip absolute bottom-full mb-3 p-3 bg-gray-800 text-white rounded-md shadow-2xl text-center text-sm">
                        <p class="font-bold text-lg mb-1">${segment.name}</p>
                        <p>Duration: <span class="font-mono">${formatTime(duration)}</span></p>
                        <p>Starts at: <span class="font-mono">${timeStartLabel}</span></p>
                        <p>Ends at: <span class="font-mono">${timeEndLabel}</span></p>
                    </div>
                `;
                
                innerWrapper.appendChild(segmentElement);
            });
            
            timelineChart.appendChild(innerWrapper);

            // Re-center the meridian line on the inner wrapper's coordinates
            const meridianLine = document.querySelector('.meridian-line');
            meridianLine.style.left = `${meridianOffsetPixels}px`;

            // Calculate the necessary scroll position to center the meridian line
            const scrollCenterPosition = meridianOffsetPixels - (containerWidth / 2);
            timelineChart.scrollLeft = scrollCenterPosition;
        }

        // --- Pan and Zoom Event Handlers for Time Axis ---
        function setupPanZoom() {
            const chart = document.getElementById('time-axis-chart');
            if (!chart) return; // Add check for safety

            // Mouse Down (Start Drag)
            chart.addEventListener('mousedown', (e) => {
                isDragging = true;
                chart.style.cursor = 'grabbing';
                startX = e.pageX - chart.offsetLeft;
                startScrollLeft = chart.scrollLeft;
                e.preventDefault();
            });

            // Mouse Leave/Up (Stop Drag)
            document.addEventListener('mouseup', () => {
                isDragging = false;
                chart.style.cursor = 'grab';
            });

            // Mouse Move (Dragging)
            chart.addEventListener('mousemove', (e) => {
                if (!isDragging) return;
                e.preventDefault();
                const x = e.pageX - chart.offsetLeft;
                const walk = (x - startX) * 1.5; // Pan speed multiplier
                chart.scrollLeft = startScrollLeft - walk;
            });

            // Scroll (Zoom)
            chart.addEventListener('wheel', (e) => {
                e.preventDefault();

                const oldScale = axisCurrentScale;
                const zoomFactor = -e.deltaY * 0.001;
                
                // Calculate new scale, clamping between 0.2 and 10.0
                axisCurrentScale = Math.max(0.2, Math.min(10.0, axisCurrentScale + zoomFactor));
                
                // Get mouse position relative to chart
                const rect = chart.getBoundingClientRect();
                const mouseX = e.clientX - rect.left;

                // Calculate the pixel position on the currently scaled timeline that is under the cursor
                const pointToZoomOn = chart.scrollLeft + mouseX;

                // Recalculate and re-render the timeline
                calculateFlip(false); // Re-render without resetting currentScale

                // Calculate the new scroll position to keep the pointToZoomOn fixed
                const timeAxisWrapper = document.getElementById('time-axis-wrapper');
                const newWidth = timeAxisWrapper ? timeAxisWrapper.scrollWidth : 0;
                
                if (newWidth === 0) return;

                const ratio = newWidth / (newWidth / oldScale * axisCurrentScale);
                const newScrollLeft = pointToZoomOn * ratio - mouseX;

                // Apply the new scroll position
                chart.scrollLeft = newScrollLeft;
            });
        }
        
        /**
         * Calculates the total downtime and generates the timeline data points.
         */
        function calculateFlip(resetScale = true) {
            // Reset axis scale only on input changes, not during zoom
            if (resetScale) {
                axisCurrentScale = 1.0;
            }

            // 1. Get User Inputs (converted to seconds where necessary)
            // Use local variables for elements to ensure single access point
            const t1Element = document.getElementById('t1');
            const t2Element = document.getElementById('t2');
            const tPauseElement = document.getElementById('t_pause');
            const tEndExpElement = document.getElementById('t_end_exp');
            const tMountLimitElement = document.getElementById('t_mount_limit');
            const tEndExpDisplayElement = document.getElementById('t_end_exp_display');
            const summaryElement = document.getElementById('summary');
            const totalTimeMinElement = document.getElementById('total-time-min');
            
            // Safety checks for critical elements
            if (!t1Element || !tEndExpElement || !tEndExpDisplayElement || !summaryElement || !totalTimeMinElement) {
                // If any critical element is missing, stop calculation
                console.error("Critical element missing, stopping calculation.");
                return;
            }


            const T1_min = parseFloat(t1Element.value) || 0;
            const T2_min = parseFloat(t2Element.value) || 0;
            const T_Pause_min = parseFloat(tPauseElement.value) || 0;
            const T_End_Exp = parseFloat(tEndExpElement.value) || 0; // Scenario slider value
            const T_MountLimit_min = parseFloat(tMountLimitElement.value) || 0; 

            // Update slider display
            tEndExpDisplayElement.textContent = formatTime(T_End_Exp);

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
            
            // --- 2. Determine Tracking Stop Time (t_StopTracking) ---
            if (T_Pause_min > 0) {
                // Scenario 1: Forced Pause is set. Tracking stops at -T_Pause_sec.
                t_DowntimeStart = -T_Pause_sec;
                scenarioDescription = "Forced Pause: Tracking stopped early to avoid an obstruction.";
            } else if (T_End_Exp > T_Effective_Deadline) {
                // Scenario 2: Exposure ends AFTER the deadline. NINA would abort/skip the last sub and flip at the deadline.
                t_DowntimeStart = T_Effective_Deadline - 1; 
                t_FlipStart = T_Effective_Deadline;
                T_Wait = 1; // Minimal wait time to force the flip at the deadline
                scenarioDescription = "Deadline Reached: Last exposure aborted/skipped. Flip forced at deadline.";
            } else {
                // Scenario 3: Normal case. Tracking stops when the last successful exposure finishes.
                t_DowntimeStart = T_End_Exp;
                scenarioDescription = `Tracking Stopped: Last exposure ended at ${formatTime(T_End_Exp)}.`;
            }
            
            // --- 3. Determine Flip Start Time (t_FlipStart) and Wait Time (T_Wait) ---

            if (scenarioDescription.includes("Deadline Reached")) {
                // t_FlipStart is already T_Effective_Deadline from Scenario 2
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
            summaryElement.innerHTML = summaryHTML;
            totalTimeMinElement.textContent = formatTime(T_Downtime);
            
            // --- 6. Render Visualization ---
            renderTimeline(segments, minTime, totalTimeSpan);
            renderTimeAxis(t_DowntimeStart, t_FlipStart, T_Effective_Deadline);
        }

        // Initialize the calculation and event listeners on page load
        document.addEventListener('DOMContentLoaded', () => {
            const tEndExpSlider = document.getElementById('t_end_exp');
            
            // Check if the slider element exists before trying to set its value
            if (tEndExpSlider) {
                // Set the slider's initial value to 0s to center it on meridian
                tEndExpSlider.value = 0; 
            } else {
                console.error("Initialization error: 't_end_exp' slider element not found.");
            }

            // Setup Pan/Zoom events for the TIME AXIS CHART
            setupPanZoom();

            calculateFlip();
            // Recalculate on resize to maintain responsiveness
            window.addEventListener('resize', calculateFlip); 
        });
    </script>
</body>
</html>
