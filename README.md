<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- Đổi tên title --><title>Green Energy Game</title>
    <!-- Tải Tailwind CSS --><script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Đảm bảo chiều cao đầy đủ */
        html, body {
            height: 100%;
            margin: 0;
            font-family: 'Inter', sans-serif;
            overflow: hidden; /* Tránh cuộn trang */
        }
        
        #game-container {
            display: flex;
            height: 100vh;
            width: 100vw;
        }

        /* Màn hình A (Khu vực mô hình) */
        #model-a {
            background-color: #6B7280; /* Gray-500 (Xám đậm mặc định - Khi chưa có mặt trời) */
            background-size: cover;
            background-position: center;
            position: relative; /* Quan trọng để định vị các item con */
            transition: background-color 0.5s ease;
        }
        
        /* CẬP NHẬT: Nền xanh lá khi có mặt trời và không bị che */
        #model-a.sun-present-bg {
            background-color: #D1FAE5; /* Green-100 (Xanh lá sáng) */
        }
        
        /* THÊM MỚI: Nền khi mặt trời bị mây che */
        #model-a.sky-partly-cloudy {
             background-color: #A3E6B4; /* Xanh lá nhạt pha xám (tùy chỉnh) */
        }


        /* Lớp SVG để vẽ dây dẫn (Nằm dưới các công cụ) */
        #wire-svg-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 10; /* Nằm dưới các công cụ (z-index 20) */
            pointer-events: none; /* Cho phép click xuyên qua */
        }
        
        /* Dây tạm thời khi đang kéo */
        #temp-wire {
            stroke: #FDE047; /* Vàng sáng */
            stroke-width: 4;
            stroke-dasharray: 8, 8;
            fill: none;
        }
        
        /* Dây vĩnh viễn sau khi thả */
        .permanent-wire {
            stroke: #22C55E; /* Green-500 (Màu xanh lá) */
            stroke-width: 5;
            fill: none;
            stroke-linecap: round;
            pointer-events: none;
        }
        
        /* CSS CHO HIỆU ỨNG DÂY DẪN "HOẠT ĐỘNG" (NÉT ĐỨT CHẠY) */
        @keyframes flow-animation {
            from { stroke-dashoffset: 0; }
            to { stroke-dashoffset: 20; } /* Điều chỉnh tốc độ chạy */
        }
        
        /* Màu vàng cam mặc định cho dây active (nguồn) */
        .permanent-wire.wire-active {
            stroke: #F59E0B; /* Màu vàng cam */
            stroke-dasharray: 10, 10; /* Nét đứt */
            /* Animation mặc định chạy từ END về START */
            animation: flow-animation 1s linear infinite;
        }
        
        /* CẬP NHẬT: Màu đỏ cho dây đèn active */
        .permanent-wire.light-wire-active {
             stroke: #EF4444; /* Red-500 */
        }
        
        /* Đảo ngược hướng chạy (thành START về END) */
        .permanent-wire.wire-active.wire-reversed {
            animation-direction: reverse;
        }
        
        /* Màn hình B (Hộp công cụ) */
        #tools-b {
            background-color: #78350F; /* Màu gỗ tối (yellow-800) */
            border-left: 8px solid #451A03; /* Viền đậm hơn */
            box-shadow: -10px 0 20px rgba(0,0,0,0.3) inset;
        }

        /* Ô chứa công cụ */
        .tool-cell {
            background-color: #92400E; /* Màu gỗ sáng hơn (yellow-700) */
            border-radius: 8px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.2) inset;
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative; /* Cần cho dấu cấm */
        }
        
        /* CSS cho chế độ Dây dẫn */
        body.wiring-mode {
            cursor: crosshair; /* Hình dấu + */
        }

        /* Ô công cụ khi được chọn (Dây dẫn) */
        .tool-cell.active-tool {
            background-color: #34D399; /* Green-400 */
            box-shadow: 0 0 15px #10B981 inset; /* Hiệu ứng phát sáng */
        }

        /* Công cụ (trong hộp) */
        .tool-item {
            width: 80px;
            height: 80px;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: grab;
            transition: transform 0.2s ease;
            font-size: 50px; /* Kích thước cho Emoji */
            position: relative; /* Cần cho dấu cấm */
        }
        
        .tool-item:hover {
            transform: scale(1.1);
        }
        
        .tool-item .svg-icon {
            width: 100%;
            height: 100%;
        }

        /* THÊM MỚI: Dấu cấm cho công cụ đã dùng */
        .tool-item.tool-prohibited {
            cursor: not-allowed; /* Đổi con trỏ */
            opacity: 0.6; /* Làm mờ đi */
        }
        .tool-item.tool-prohibited::before {
            content: '';
            position: absolute;
            top: 5px;
            left: 5px;
            width: 20px;
            height: 20px;
            background-color: #EF4444; /* Red-500 */
            border-radius: 50%;
            border: 2px solid white;
            z-index: 1; /* Nằm trên công cụ */
        }
         .tool-item.tool-prohibited::after {
            content: '';
            position: absolute;
            top: 13px; /* Điều chỉnh vị trí gạch chéo */
            left: 7px;
            width: 16px;
            height: 4px; /* Độ dày gạch chéo */
            background-color: white;
            transform: rotate(45deg);
            z-index: 2; /* Nằm trên vòng tròn */
        }


        /* Công cụ đã được thả (trên màn hình A) */
        .cloned-item {
            position: absolute;
            z-index: 20;
            width: 80px;
            height: 80px;
            font-size: 50px;
            cursor: grab;
            display: flex;
            justify-content: center;
            align-items: center;
            /* Thêm transition để di chuyển mượt mà khi "snap" */
            transition: left 0.3s ease, top 0.3s ease;
        }
        
        .cloned-item .svg-icon {
            width: 100%;
            height: 100%;
        }
        
        .cloned-item:active {
            cursor: grabbing;
            transition: none; /* Tắt transition khi đang kéo */
        }

        /* Trạng thái khi đang kéo (cả bản sao và bản gốc) */
        .dragging {
            position: absolute; /* Phải là absolute để di chuyển tự do */
            z-index: 1000; /* Luôn nổi lên trên cùng */
            pointer-events: none; /* Không cản trở sự kiện mouseup */
            opacity: 0.9;
            transition: none; /* Tắt transition khi đang kéo */
        }

        /* CSS CHO ANIMATION QUAY */
        @keyframes spin {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }

        /* Đặt tâm quay cho các cánh quạt (dựa trên viewBox 100x100, tâm ở 50, 35) */
        .turbine-blades {
            transform-origin: 50% 35%;
        }

        /* Áp dụng animation khi item có class 'spinning' */
        .cloned-item.spinning .turbine-blades {
            animation: spin 2s linear infinite;
        }
        
        /* === CSS PIN MẶT TRỜI === */
        
        /* Kiểu cell pin mặt trời (màu xanh dương đậm) */
        .solar-cell {
            fill: #1E3A8A; /* blue-900 (Xanh dương đậm) */
            transition: fill 0.2s ease;
        }

        /* Animation nhấp nháy Vàng <-> Xanh */
        @keyframes solar-blink {
            0%, 100% { fill: #1E3A8A; } /* Xanh dương đậm */
            50% { fill: #F59E0B; }      /* Vàng cam (giống màu dây) */
        }

        /* Khi mặt trời active, áp dụng animation cho các cell */
        .cloned-item.solar-active .solar-cell {
            /* step-end giúp animation "nhảy" màu, không bị mờ */
            animation: solar-blink 1s infinite step-end;
        }

        /* Tạo hiệu ứng "tự do" (nhấp nháy ngẫu nhiên) bằng cách delay khác nhau */
        .cloned-item.solar-active .solar-cell:nth-child(3n+1) {
            animation-delay: -0.2s;
        }
        .cloned-item.solar-active .solar-cell:nth-child(3n+2) {
            animation-delay: -0.5s;
        }
        .cloned-item.solar-active .solar-cell:nth-child(3n+3) {
            animation-delay: -0.8s;
        }

        /* === CSS BÓNG ĐÈN VÀ CÔNG TẮC === */
        
        /* Phần thủy tinh của bóng đèn (mặc định màu xám) */
        .bulb-glass {
            fill: #6B7280; /* Gray-500 */
            transition: fill 0.3s ease;
        }
        
        /* Khi đèn bật, chuyển màu vàng */
        .cloned-item.bulb-on .bulb-glass {
            fill: #FDE047; /* Yellow-300 */
            /* Thêm hiệu ứng phát sáng nhẹ */
            filter: drop-shadow(0 0 10px #FDE047);
        }

        /* Cần gạt công tắc (mặc định TẮT - Đỏ, Vị trí dưới) */
        .switch-lever {
            fill: #EF4444; /* Red-500 */
            transition: fill 0.2s ease, y 0.2s ease;
            /* y="45" được đặt trong SVG */
        }

        /* Khi công tắc BẬT (thêm class .switch-on) */
        .cloned-item.switch-on .switch-lever {
            fill: #22C55E; /* Green-500 */
            /* Di chuyển cần gạt lên trên (thay đổi thuộc tính y) */
            animation: switch-on-anim 0.2s forwards;
        }
        
        /* Cần animation để thay đổi 'y' vì nó là thuộc tính SVG, không phải CSS */
        @keyframes switch-on-anim {
            from { y: 45; }
            to { y: 25; }
        }
        
        /* Chữ (cả ON và OFF) */
        .switch-text {
            font-size: 14px;
            font-weight: bold;
            font-family: Arial, sans-serif;
            text-anchor: middle;
            user-select: none; /* Không cho phép chọn chữ */
        }

        /* Chữ OFF (Đỏ, ở trên, mặc định Bật) */
        .switch-text-off {
            fill: #EF4444;
            display: block;
        }

        /* Chữ ON (Xanh, ở dưới, mặc định Tắt) */
        .switch-text-on {
            fill: #22C55E;
            display: none;
        }

        /* Khi BẬT, hiện chữ ON, ẩn chữ OFF */
        .cloned-item.switch-on .switch-text-on {
            display: block;
        }
        .cloned-item.switch-on .switch-text-off {
            display: none;
        }

        /* Bảng tên Tủ điện */
        .power-box-label {
            font-size: 8px; /* Kích thước nhỏ */
            font-weight: bold;
            font-family: Arial, sans-serif;
            fill: #FFFFFF; /* Màu trắng */
            text-anchor: middle;
            user-select: none;
        }


    </style>
</head>
<body class="bg-blue-100 h-screen w-screen">

    <div id="game-container">
        
        <!-- MÀN HÌNH A (MÔ HÌNH) --><div id="model-a" class="w-4/5 h-full">
            <!-- Lớp SVG để vẽ dây dẫn --><svg id="wire-svg-layer"></svg>
            <!-- Các công cụ được thả vào đây --></div>

        <!-- MÀN HÌNH B (HỘP CÔNG CỤ) --><div id="tools-b" class="w-1/5 h-full p-4 grid grid-cols-2 gap-4 overflow-y-auto">
            
            <!-- Hàng 1 --><div class="tool-cell">
                <div class="tool-item" data-tool="sun" title="Mặt trời">☀️</div>
            </div>
            <div class="tool-cell">
                <!-- Công cụ 2: Ngôi nhà (SVG) --><div class="tool-item" data-tool="house" title="Ngôi nhà">
                    <svg class="svg-icon" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                        <!-- Mái nhà (lớn, màu đỏ) --><path d="M 5 40 L 50 10 L 95 40 Z" fill="#DC2626" stroke="#451A03" stroke-width="2"/>
                        <!-- Thân nhà (màu vàng) --><rect x="15" y="40" width="70" height="55" fill="#FBBF24" stroke="#451A03" stroke-width="2"/>
                        <!-- Cửa --><rect id="house-door" x="40" y="60" width="20" height="35" fill="#78350F" stroke="#451A03" stroke-width="2"/>
                    </svg>
                </div>
            </div>

            <!-- Hàng 2 --><div class="tool-cell">
                <!-- CÔNG CỤ 3: PIN MẶT TRỜI (CẬP NHẬT) -->
                <div class="tool-item" data-tool="solar-panel" title="Pin mặt trời">
                    <svg class="svg-icon" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                        <g transform="skewX(-20) translate(15 0)">
                            <!-- Khung ngoài (màu sẫm) -->
                            <rect x="10" y="25" width="70" height="50" fill="#064E3B" stroke="#064E3B" stroke-width="2" />
                            
                            <!-- Lưới 3x4 (12 cell) -->
                            <g>
                                <!-- Hàng 1 -->
                                <rect class="solar-cell" x="10.5" y="25.5" width="22.6" height="11.75"/>
                                <rect class="solar-cell" x="33.6" y="25.5" width="22.6" height="11.75"/>
                                <rect class="solar-cell" x="56.9" y="25.5" width="22.6" height="11.75"/>
                                <!-- Hàng 2 -->
                                <rect class="solar-cell" x="10.5" y="37.75" width="22.6" height="11.75"/>
                                <rect class="solar-cell" x="33.6" y="37.75" width="22.6" height="11.75"/>
                                <rect class="solar-cell" x="56.9" y="37.75" width="22.6" height="11.75"/>
                                <!-- Hàng 3 -->
                                <rect class="solar-cell" x="10.5" y="50" width="22.6" height="11.75"/>
                                <rect class="solar-cell" x="33.6" y="50" width="22.6" height="11.75"/>
                                <rect class="solar-cell" x="56.9" y="50" width="22.6" height="11.75"/>
                                <!-- Hàng 4 -->
                                <rect class="solar-cell" x="10.5" y="62.25" width="22.6" height="11.75"/>
                                <rect class="solar-cell" x="33.6" y="62.25" width="22.6" height="11.75"/>
                                <rect class="solar-cell" x="56.9" y="62.25" width="22.6" height="11.75"/>
                            </g>
                            
                            <!-- Đường kẻ lưới màu trắng -->
                            <g stroke="#FFF" stroke-width="0.75">
                                <!-- Dọc -->
                                <line x1="33.35" y1="25" x2="33.35" y2="75" />
                                <line x1="56.65" y1="25" x2="56.65" y2="75" />
                                <!-- Ngang -->
                                <line x1="10" y1="37.5" x2="80" y2="37.5" />
                                <line x1="10" y1="50" x2="80" y2="50" />
                                <line x1="10" y1="62.5" x2="80" y2="62.5" />
                            </g>
                        </g>
                    </svg>
                </div>
            </div>
            <div class="tool-cell">
                <!-- Công cụ 4: Tua-bin gió (SVG 3 cánh quạt) --><div class="tool-item" data-tool="wind-turbine" title="Tua-bin gió">
		    	<svg class="svg-icon" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
		            <!-- Trụ --><path d="M48 40 L48 95 Q 48 100 45 100 L 55 100 Q 52 100 52 95 L 52 40 Z" fill="#9CA3AF"/>
		            <!-- Nhóm cánh quạt (để xoay) --><g class="turbine-blades">
		            <!-- Cánh 1 --><path d="M 50 35 L 65 20 L 70 25 L 55 40 Z" fill="#F3F4F6" stroke="#9CA3AF" stroke-width="1.5" transform="rotate(0 50 35)"/>
            		    <!-- Cánh 2 --><path d="M 50 35 L 65 20 L 70 25 L 55 40 Z" fill="#F3F4F6" stroke="#9CA3AF" stroke-width="1.5" transform="rotate(120 50 35)"/>
            		    <!-- Cánh 3 --><path d="M 50 35 L 65 20 L 70 25 L 55 40 Z" fill="#F3F4F6" stroke="#9CA3AF" stroke-width="1.5" transform="rotate(240 50 35)"/>
        		</g>
        		<!-- Tâm --><circle cx="50" cy="35" r="7" fill="#6B7280" stroke="#4B5563" stroke-width="2"/>
    		    </svg>
		</div>
            </div>

            <!-- Hàng 3 --><div class="tool-cell">
                <!-- CÔNG CỤ 5: CÔNG TẮC (CẬP NHẬT) -->
                <div class="tool-item" data-tool="switch" title="Công tắc">
                    <svg class="svg-icon" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                        <rect x="20" y="20" width="60" height="60" rx="10" fill="#E5E7EB" stroke="#4B5563" stroke-width="3"/>
                        
                        <!-- Chữ OFF (ở trên) -->
                        <text x="50" y="38" class="switch-text switch-text-off">OFF</text>
                        
                        <!-- Cần gạt (Vị trí Y được điều khiển bằng CSS) -->
                        <rect class="switch-lever" x="40" y="45" width="20" height="30" rx="5"/>

                        <!-- Chữ ON (ở dưới) -->
                        <text x="50" y="70" class="switch-text switch-text-on">ON</text>
                    </svg>
                </div>
            </div>
            <div class="tool-cell">
                <!-- CÔNG CỤ 6: BÓNG ĐÈN (CẬP NHẬT) -->
                <div class="tool-item" data-tool="light-bulb" title="Bóng đèn">
                    <svg class="svg-icon" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                         <!-- Đuôi đèn -->
                        <rect x="35" y="60" width="30" height="20" fill="#9CA3AF" stroke="#4B5563" stroke-width="2"/>
                        <rect x="38" y="80" width="24" height="5" fill="#6B7280"/>
                         <!-- Phần thủy tinh (TẮT, màu xám) -->
                        <circle class="bulb-glass" cx="50" cy="40" r="30" stroke="#4B5563" stroke-width="2"/>
                    </svg>
                </div>
            </div>
            
            <!-- Hàng 4 --><div class="tool-cell">
                <!-- Công cụ 7: Tủ điện (CẬP NHẬT: Thêm bảng tên) --><div class="tool-item" data-tool="power-box" title="Tủ điện">
                    <svg class="svg-icon" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                        <rect x="20" y="15" width="60" height="70" rx="8" fill="#6B7280" stroke="#374151" stroke-width="3"/>
                        <rect x="28" y="25" width="44" height="50" rx="4" fill="#D1D5DB" stroke="#4B5563" stroke-width="2"/>
                        <circle cx="38" cy="50" r="5" fill="#EF4444"/>
                        <circle cx="62" cy="50" r="5" fill="#22C55E"/>
                        <!-- Bảng tên -->
                        <text x="50" y="80" class="power-box-label">Tủ điện</text>
                    </svg>
                </div>
            </div>
            <div class="tool-cell" id="wire-tool-cell">
                <!-- Công cụ 8: Dây dẫn điện (SVG) - Nút kích hoạt chế độ --><div class="tool-item" data-tool="wire" title="Dây dẫn điện">
                    <svg class="svg-icon" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                        <path d="M 20 20 Q 50 80 80 80" stroke="#F59E0B" stroke-width="8" fill="none" stroke-linecap="round"/>
                        <circle cx="20" cy="20" r="10" fill="#3B82F6"/>
                        <circle cx="80" cy="80" r="10" fill="#3B82F6"/>
                    </svg>
                </div>
            </div>
            
            <!-- Hàng 5 --><div class="tool-cell">
                <div class="tool-item" data-tool="cloud" title="Đám mây">☁️</div>
            </div>
            <div class="tool-cell">
                <div class="tool-item" data-tool="wind" title="Luồng gió thổi">💨</div>
            </div>

            <!-- Hàng 6 (Trống) --><div class="tool-cell"></div>
            <div class="tool-cell"></div>
        </div>
    </div>

    <script>
        const dropZone = document.getElementById('model-a');
        const toolBox = document.getElementById('tools-b');
        const wireToolCell = document.getElementById('wire-tool-cell');
        const wireLayer = document.getElementById('wire-svg-layer');

        let draggedItem = null;      
        let draggedItemCopy = null; 
        let offsetX, offsetY;
        
        // SỬA LỖI CLICK CÔNG TẮC: Biến theo dõi trạng thái kéo
        let isDragging = false; // Theo dõi nếu đang kéo (kể cả clone hoặc move)
        let isPotentialDrag = false; // Theo dõi nếu nhấn chuột (có thể là click hoặc drag)
        
        let isWiringMode = false; 

        // WIRING VARIABLES
        let wiringStartItem = null;
        let tempWire = null;
        // Danh sách các công cụ có thể tham gia nối dây
        const wiringParticipants = ['power-box', 'solar-panel', 'wind-turbine', 'light-bulb', 'switch'];
        // DANH SÁCH CÔNG CỤ ĐỘC QUYỀN (trừ Mây và Dây)
        const exclusiveTools = ['sun', 'house', 'solar-panel', 'wind-turbine', 'switch', 'light-bulb', 'power-box', 'wind'];
        
        // BIẾN TRẠNG THÁI TOÀN CỤC (QUAN TRỌNG)
        let isSunnyGlobal = false; // Trạng thái CÓ NẮNG (Mặt trời có VÀ không bị che)


        
        // === HÀM HỖ TRỢ ===
        function generateUUID() {
            return 'item-' + Date.now() + '-' + Math.random().toString(16).slice(2);
        }

        // Lấy tọa độ chuột tương đối với dropZone
        function getMousePos(evt) {
            const rect = dropZone.getBoundingClientRect();
            return {
                x: evt.clientX - rect.left,
                y: evt.clientY - rect.top
            };
        }
        
        // Lấy tọa độ tâm của một công cụ (tương đối với dropZone)
        function getCenter(element) {
            const rect = element.getBoundingClientRect();
            const dropZoneRect = dropZone.getBoundingClientRect();
            // Trả về trung tâm hình học
            return {
                x: (rect.left + rect.right) /2 - dropZoneRect.left,
                y: (rect.top + rect.bottom) / 2 - dropZoneRect.top
            };
        }
        
        // Lấy tọa độ điểm nối ở đáy (tương đối với dropZone)
        function getBottomCenter(element) {
            const rect = element.getBoundingClientRect();
            const dropZoneRect = dropZone.getBoundingClientRect();
            // Trả về trung tâm x, cạnh đáy y
            return {
                x: (rect.left + rect.right) / 2 - dropZoneRect.left,
                y: rect.bottom - dropZoneRect.top 
            };
        }

        // Lấy tọa độ điểm nối ở đỉnh (tương đối với dropZone)
        function getTopCenter(element) {
            const rect = element.getBoundingClientRect();
            const dropZoneRect = dropZone.getBoundingClientRect();
            // Trả về trung tâm x, cạnh đỉnh y
            return {
                x: (rect.left + rect.right) / 2 - dropZoneRect.left,
                y: rect.top - dropZoneRect.top 
            };
        }
        
        // Lấy tọa độ điểm nối ở cạnh bên gần nhất (tương đối với dropZone)
        function getClosestSideCenter(element, targetElement) {
            const rect = element.getBoundingClientRect();
            const dropZoneRect = dropZone.getBoundingClientRect();
            const targetCenter = getCenter(targetElement);
            const elementCenter = getCenter(element);

            let xPos;
            
            // Nếu element (Item) nằm bên trái target (PBox), dùng cạnh phải của Item.
            if (elementCenter.x < targetCenter.x) {
                xPos = rect.right - dropZoneRect.left;
            } else {
                // Nếu element (Item) nằm bên phải target (PBox), dùng cạnh trái của Item.
                xPos = rect.left - dropZoneRect.left;
            }
            
            // Trả về trung tâm y, cạnh x gần nhất
            return {
                x: xPos,
                y: elementCenter.y
            };
        }
        
        // HÀM HỖ TRỢ TÍNH TOÁN OVERLAP (AABB)
        function getOverlap(rect1, rect2) {
            const overlapLeft = Math.max(rect1.left, rect2.left);
            const overlapRight = Math.min(rect1.right, rect2.right);
            const overlapTop = Math.max(rect1.top, rect2.top);
            const overlapBottom = Math.min(rect1.bottom, rect2.bottom);

            if (overlapRight > overlapLeft && overlapBottom > overlapTop) {
                return (overlapRight - overlapLeft) * (overlapBottom - overlapTop);
            }
            return 0; // No overlap
        }


        // === HÀM QUẢN LÝ DÂY DẪN ===

        /**
         * Xóa tất cả các dây dẫn được nối với một công cụ (dựa trên itemId)
         * @param {string} itemId ID của công cụ bị xóa.
         */
        function removeWires(itemId) {
            const wires = Array.from(wireLayer.querySelectorAll('.permanent-wire'));
            wires.forEach(wire => {
                if (wire.dataset.startId === itemId || wire.dataset.endId === itemId) {
                    wire.remove();
                }
            });
        }
        
        /**
         * KIỂM TRA XEM CÔNG TẮC/ĐÈN CÓ ĐƯỢC NỐI DÂY KHÔNG
         */
        function isItemWired(item) {
            if (!item) return false;
            const itemId = item.dataset.itemId;
            const wire = wireLayer.querySelector(`.permanent-wire[data-start-id="${itemId}"], .permanent-wire[data-end-id="${itemId}"]`);
            return !!wire; // Trả về true nếu tìm thấy dây, ngược lại false
        }

        /**
         * Tính toán đường đi (path D) cho dây dẫn dựa trên loại công cụ
         * @param {Element} startEl 
         * @param {Element} endEl 
         * @returns {string} Chuỗi d của SVG path
         */
        function calculateWirePath(startEl, endEl) {
            // Thêm kiểm tra null phòng trường hợp 1 trong 2 bị thiếu
            if (!startEl || !endEl) return ""; 

            const startType = startEl.dataset.tool;
            const endType = endEl.dataset.tool;
            let pathD;

            // KIỂM TRA KẾT NỐI ĐẶC BIỆT
            const isTurbineConnection = (startType === 'power-box' && endType === 'wind-turbine') || 
                                       (startType === 'wind-turbine' && endType === 'power-box');
            
            const isSolarConnection = (startType === 'power-box' && endType === 'solar-panel') || 
                                      (startType === 'solar-panel' && endType === 'power-box');
                                      
            const isSwitchConnection = (startType === 'power-box' && endType === 'switch') ||
                                       (startType === 'switch' && endType === 'power-box');
                                       
            const isLightBulbConnection = (startType === 'power-box' && endType === 'light-bulb') ||
                                          (startType === 'light-bulb' && endType === 'power-box');

            if (isTurbineConnection) {
                // LOGIC NỐI DÂY TUA-BIN GÓC VUÔNG (Tủ điện <-> Tua-bin)
                const PBox = startType === 'power-box' ? startEl : endEl;
                const Turbine = startType === 'wind-turbine' ? startEl : endEl;
                const S = getBottomCenter(PBox); 
                const E = getBottomCenter(Turbine);
                const commonY = Math.max(S.y, E.y) + 15; // Điểm chung dưới "mặt đất"
                pathD = `M ${S.x} ${S.y} L ${S.x} ${commonY} L ${E.x} ${commonY} L ${E.x} ${E.y}`;
                
            } else if (isSolarConnection) {
                 // LOGIC NỐY DÂY PIN MẶT TRỜI GÓC VUÔNG (Horizontal then Vertical)
                const PBox = startType === 'power-box' ? startEl : endEl;
                const Solar = startType === 'solar-panel' ? startEl : endEl;
                const P1 = getClosestSideCenter(Solar, PBox); // Cạnh bên Pin mặt trời
                const P4 = getTopCenter(PBox); // Đỉnh Tủ điện
                const P2 = { x: P4.x, y: P1.y }; // Điểm rẽ góc 90 độ
                pathD = `M ${P1.x} ${P1.y} L ${P2.x} ${P2.y} L ${P4.x} ${P4.y}`;

            } else if (isSwitchConnection) {
                // CẬP NHẬT: LOGIC NỐI DÂY CÔNG TẮC (Giống Tua-bin)
                const PBox = startType === 'power-box' ? startEl : endEl;
                const Switch = startType === 'switch' ? startEl : endEl;
                const S = getBottomCenter(PBox); 
                const E = getBottomCenter(Switch);
                const commonY = Math.max(S.y, E.y) + 15; // Điểm chung dưới "mặt đất"
                pathD = `M ${S.x} ${S.y} L ${S.x} ${commonY} L ${E.x} ${commonY} L ${E.x} ${E.y}`;
                
            } else if (isLightBulbConnection) {
                // LOGIC NỐI DÂY BÓNG ĐÈN GÓC VUÔNG (Horizontal then Vertical)
                const PBox = startType === 'power-box' ? startEl : endEl;
                const LightBulb = startType === 'light-bulb' ? startEl : endEl;
                const P1 = getBottomCenter(LightBulb); // Đáy Bóng đèn
                const P4 = getTopCenter(PBox); // Đỉnh Tủ điện
                const P2 = { x: P4.x, y: P1.y }; // Điểm rẽ góc 90 độ
                pathD = `M ${P1.x} ${P1.y} L ${P2.x} ${P2.y} L ${P4.x} ${P4.y}`;

            } else {
                // LOGIC MẶC ĐỊNH cho các kết nối khác (Bezier curve)
                const startPos = getCenter(startEl);
                const endPos = getCenter(endEl);
                pathD = `M ${startPos.x} ${startPos.y} Q ${startPos.x} ${endPos.y} ${endPos.x} ${endPos.y}`;
            }
            
            return pathD;
        }

        // VẼ DÂY VĨNH VIỄN
        function createPermanentWire(startEl, endEl) {
            const startId = startEl.dataset.itemId;
            const endId = endEl.dataset.itemId;
            
            const pathD = calculateWirePath(startEl, endEl);
            
            const wire = document.createElementNS('http://www.w3.org/2000/svg', 'path');
            wire.setAttribute('class', 'permanent-wire');
            wire.dataset.startId = startId;
            wire.dataset.endId = endId;
            wire.setAttribute('d', pathD);
            
            wireLayer.prepend(wire); 
        }

        /**
         * Cập nhật tất cả các dây dẫn được nối với một công cụ đang di chuyển
         * @param {string} itemId ID của công cụ đang di chuyển
         */
        function updateConnectedWires(itemId) {
            const wires = Array.from(wireLayer.querySelectorAll('.permanent-wire'));
            const movingItem = document.querySelector(`[data-item-id="${itemId}"]`);
            if (!movingItem) return;

            wires.forEach(wire => {
                let startEl, endEl;
                
                if (wire.dataset.startId === itemId) {
                    startEl = movingItem;
                    endEl = document.querySelector(`[data-item-id="${wire.dataset.endId}"]`);
                } else if (wire.dataset.endId === itemId) {
                    startEl = document.querySelector(`[data-item-id="${wire.dataset.startId}"]`);
                    endEl = movingItem;
                } else {
                    return; // Bỏ qua nếu dây không liên quan
                }

                if (startEl && endEl) {
                    const newPathD = calculateWirePath(startEl, endEl);
                    wire.setAttribute('d', newPathD);
                }
            });
        }
        
        // === HÀM QUẢN LÝ TRẠNG THÁI MÔ PHỎNG ===
        
        /**
         * HÀM CHỦ: Cập nhật toàn bộ trạng thái mô phỏng
         * Đây là hàm điều khiển chính, gọi các hàm con theo thứ tự.
         */
        function updateAllSimulationStates() {
            // 1. Cập nhật trạng thái có nắng (Mặt trời có & không bị che) và màu nền
            isSunnyGlobal = checkSunAndCloudsAndUpdateBackground();
            
            // 2. Cập nhật Pin mặt trời (dựa vào isSunnyGlobal)
            updateSolarPanelState(isSunnyGlobal);
            
            // 3. Cập nhật Gió (Tua-bin quay)
            updateWindAnimation();
            
            // 4. Cập nhật Đèn (dựa vào Pin, Gió, Công tắc)
            updateLightBulbState();
            
            // 5. Cập nhật Dây dẫn (dựa vào Pin, Gió, VÀ TRẠNG THÁI ĐÈN)
            updatePowerGrid(); 
            
            // 6. Cập nhật trạng thái Hộp công cụ
            updateToolboxAvailability();
        }
        
        /**
         * 1. Kiểm tra Mặt trời, Mây che, cập nhật nền trời và trạng thái isSunnyGlobal
         * @returns {boolean} Trả về true nếu CÓ NẮNG (Mặt trời có VÀ không bị che)
         */
        function checkSunAndCloudsAndUpdateBackground() {
            const sun = document.querySelector('#model-a [data-tool="sun"]');
            const clouds = document.querySelectorAll('#model-a [data-tool="cloud"]');
            
            let isSunny = false; // Mặc định là không có nắng
            
            if (sun) {
                // CÓ công cụ mặt trời
                
                // Kiểm tra mây che
                const sunRect = sun.getBoundingClientRect();
                const sunArea = (sunRect.right - sunRect.left) * (sunRect.bottom - sunRect.top);
                let totalOverlap = 0;

                clouds.forEach(cloud => {
                    const cloudRect = cloud.getBoundingClientRect();
                    totalOverlap += getOverlap(sunRect, cloudRect);
                });

                // Nếu không bị che quá 50% -> CÓ NẮNG và nền xanh lá sáng
                if (totalOverlap <= (sunArea * 0.5)) {
                    isSunny = true;
                    dropZone.classList.add('sun-present-bg');
                    dropZone.classList.remove('sky-partly-cloudy');
                } else {
                    // Bị che -> KHÔNG có nắng và nền xanh lá pha xám
                    isSunny = false;
                    dropZone.classList.remove('sun-present-bg');
                    dropZone.classList.add('sky-partly-cloudy');
                }
                
            } else {
                // KHÔNG có công cụ mặt trời -> Nền xám đậm (mặc định) và không có nắng
                dropZone.classList.remove('sun-present-bg');
                dropZone.classList.remove('sky-partly-cloudy');
                isSunny = false; // Không có nắng
            }
            
            return isSunny; // Trả về trạng thái có nắng
        }
        
        /**
         * 2. Cập nhật trạng thái Pin mặt trời (nhấp nháy)
         * @param {boolean} isSunny Trạng thái có nắng (đã tính toán)
         */
        function updateSolarPanelState(isSunny) {
            const solarPanels = document.querySelectorAll('#model-a [data-tool="solar-panel"]');

            solarPanels.forEach(panel => {
                // Pin chỉ active khi CÓ NẮNG
                if (isSunny) {
                    panel.classList.add('solar-active');
                } else {
                    panel.classList.remove('solar-active');
                }
            });
            
            // updatePowerGrid(); // Sẽ được gọi trong hàm updateAllSimulationStates
        }

        /**
         * 3. CẬP NHẬT ANIMATION GIÓ
         */
        function updateWindAnimation() {
            const windIsPresent = !!document.querySelector('#model-a [data-tool="wind"]');
            const turbines = document.querySelectorAll('#model-a [data-tool="wind-turbine"]');

            turbines.forEach(turbine => {
                if (windIsPresent) {
                    turbine.classList.add('spinning');
                } else {
                    turbine.classList.remove('spinning');
                }
            });
            
            // updatePowerGrid(); // Sẽ được gọi trong hàm updateAllSimulationStates
        }
        
        /**
         * 4. Cập nhật trạng thái Bóng đèn (Bật/Tắt)
         */
        function updateLightBulbState() {
            const lightBulbs = document.querySelectorAll('#model-a [data-tool="light-bulb"]');
            if (lightBulbs.length === 0) return; // Không có đèn, bỏ qua

            // CẬP NHẬT LOGIC: Kiểm tra nguồn CÓ DÂY
            const turbineEl = document.querySelector('#model-a [data-tool="wind-turbine"]');
            const solarPanelEl = document.querySelector('#model-a [data-tool="solar-panel"]');
            
            const isWiredTurbineSpinning = turbineEl && turbineEl.classList.contains('spinning') && isItemWired(turbineEl);
            const isWiredSolarActive = isSunnyGlobal && solarPanelEl && isItemWired(solarPanelEl);
            
            const isPowerAvailable = isWiredTurbineSpinning || isWiredSolarActive;

            // Kiểm tra công tắc
            const mainSwitch = document.querySelector('#model-a [data-tool="switch"]');
            
            let isLightOn = false;
            
            // Logic đèn sáng
            if (mainSwitch) {
                // CÓ CÔNG TẮC: Đèn sáng = Nguồn CÓ DÂY VÀ Công tắc BẬT VÀ Công tắc CÓ DÂY
                if (isPowerAvailable && 
                    mainSwitch.classList.contains('switch-on') &&
                    isItemWired(mainSwitch) ) { 
                    
                    isLightOn = true; 
                }
            } else {
                // KHÔNG CÓ CÔNG TẮC: Đèn sáng = Nguồn CÓ DÂY
                if (isPowerAvailable) {
                    isLightOn = true;
                }
            }

            // Áp dụng trạng thái cho tất cả bóng đèn (VÀ KIỂM TRA DÂY CỦA ĐÈN)
            lightBulbs.forEach(bulb => {
                // Đèn chỉ sáng nếu (isLightOn (đã check nguồn/công tắc/dây nguồn) VÀ chính nó (bóng đèn) CÓ DÂY)
                if (isLightOn && isItemWired(bulb)) {
                    bulb.classList.add('bulb-on');
                } else {
                    bulb.classList.remove('bulb-on');
                }
            });
        }
        
        /**
         * 5. Cập nhật trạng thái "hoạt động" (nét đứt) của dây dẫn
         */
        function updatePowerGrid() {
            const powerBox = document.querySelector('#model-a [data-tool="power-box"]');
            const isSunny = isSunnyGlobal; 

            if (!powerBox) {
                document.querySelectorAll('.permanent-wire').forEach(w => w.classList.remove('wire-active', 'wire-reversed', 'light-wire-active'));
                return;
            }

            const powerBoxId = powerBox.dataset.itemId;
            const allWires = document.querySelectorAll('.permanent-wire');

            allWires.forEach(wire => {
                const startId = wire.dataset.startId;
                const endId = wire.dataset.endId;
                const item1 = document.querySelector(`[data-item-id="${startId}"]`);
                const item2 = document.querySelector(`[data-item-id="${endId}"]`);

                if (!item1 || !item2) {
                    wire.classList.remove('wire-active', 'wire-reversed', 'light-wire-active');
                    return;
                }

                const item1Type = item1.dataset.tool;
                const item2Type = item2.dataset.tool;

                let isActive = false;
                 // SỬA LỖI HIỆU ỨNG DÂY DẪN (Đảo ngược)
                 // true = chạy START -> END (đảo ngược CSS), false = chạy END -> START (CSS mặc định)
                let useReverseAnimation = false;
                let isLightWire = false;

                // Case 1: Pin -> Tủ hoặc Tủ -> Pin
                if ((item1Type === 'solar-panel' && item2Type === 'power-box') || 
                    (item1Type === 'power-box' && item2Type === 'solar-panel')) {
                    if (isSunny) { 
                        isActive = true;
                        // Hướng chạy: Pin -> Tủ
                        useReverseAnimation = (item1Type === 'solar-panel'); // Reverse nếu Pin là Start
                    }
                } 
                // Case 2: Tua-bin -> Tủ hoặc Tủ -> Tua-bin
                else if ((item1Type === 'wind-turbine' && item2Type === 'power-box') || 
                           (item1Type === 'power-box' && item2Type === 'wind-turbine')) {
                    const turbineEl = (item1Type === 'wind-turbine') ? item1 : item2;
                    if (turbineEl.classList.contains('spinning')) {
                        isActive = true;
                        // Hướng chạy: Tua-bin -> Tủ
                        useReverseAnimation = (item1Type === 'wind-turbine'); // Reverse nếu Tua-bin là Start
                    }
                }
                // Case 3: Tủ -> Đèn hoặc Đèn -> Tủ
                else if ((item1Type === 'light-bulb' && item2Type === 'power-box') || 
                           (item1Type === 'power-box' && item2Type === 'light-bulb')) {
                    const lightBulbEl = (item1Type === 'light-bulb') ? item1 : item2;
                    if (lightBulbEl.classList.contains('bulb-on')) {
                        isActive = true;
                        isLightWire = true;
                        // Hướng chạy: Tủ -> Đèn
                        useReverseAnimation = (item1Type === 'power-box'); // Reverse nếu Tủ là Start
                    }
                }

                // Apply classes
                if (isActive) {
                    wire.classList.add('wire-active');
                    if (useReverseAnimation) {
                        wire.classList.add('wire-reversed');
                    } else {
                        wire.classList.remove('wire-reversed');
                    }
                    if (isLightWire) {
                        wire.classList.add('light-wire-active');
                    } else {
                        wire.classList.remove('light-wire-active');
                    }
                } else {
                    wire.classList.remove('wire-active', 'wire-reversed', 'light-wire-active');
                }
            });
        }
        
        /**
         * 6. Cập nhật trạng thái Hộp công cụ (Thêm dấu cấm)
         */
         function updateToolboxAvailability() {
             const toolboxItems = toolBox.querySelectorAll('.tool-item');
             toolboxItems.forEach(item => {
                 const toolType = item.dataset.tool;
                 // Bỏ qua dây dẫn và đám mây
                 if (toolType === 'wire' || toolType === 'cloud') {
                     item.classList.remove('tool-prohibited'); // Đảm bảo chúng không bao giờ bị cấm
                     return;
                 }
                 
                 // Kiểm tra xem công cụ có tồn tại trên màn hình A không
                 const itemExistsOnScreen = !!dropZone.querySelector(`.cloned-item[data-tool="${toolType}"]`);
                 
                 if (itemExistsOnScreen) {
                     item.classList.add('tool-prohibited');
                 } else {
                     item.classList.remove('tool-prohibited');
                 }
             });
         }

        
        // === HÀM TỰ ĐỘNG GẮN (SNAP) ===
        
        /**
         * HÀM CHỦ MỚI: Gọi tất cả các hàm snap
         */
        function updateAllSnapPositions() {
            // Dùng setTimeout để đảm bảo vị trí Ngôi nhà được cập nhật
            // trước khi gắn các vật thể khác vào
            setTimeout(() => {
                snapPowerBoxToHouse();
                snapSolarPanelToHouse();
                snapSwitchToHouse(); // THÊM MỚI
                snapLightBulbToHouse(); // THÊM MỚI
            }, 0); // 0ms delay, chỉ để đẩy vào cuối hàng đợi thực thi
        }

        /**
         * HÀM SNAP: Tự động gắn Tủ điện (power-box) vào Ngôi nhà (house)
         */
        function snapPowerBoxToHouse() {
            const house = document.querySelector('#model-a [data-tool="house"]');
            const powerBox = document.querySelector('#model-a [data-tool="power-box"]');

            // Chỉ thực hiện khi cả hai tồn tại
            if (house && powerBox) {
                // Lấy kích thước và vị trí style (đã được đặt bằng px)
                const houseTop = parseFloat(house.style.top);
                const houseLeft = parseFloat(house.style.left);
                const houseHeight = parseFloat(house.style.height);
                
                const powerBoxHeight = parseFloat(powerBox.style.height);
                const powerBoxWidth = parseFloat(powerBox.style.width);

                // Tính toán vị trí mới
                // ĐIỀU CHỈNH: Gắn vào "chân mép tường" (dưới cùng bên trái)
                // Đặt Y: Gần đáy nhà (houseTop + houseHeight) trừ đi chiều cao Tủ điện, và 1 khoảng đệm nhỏ (10px)
                let targetY = houseTop + houseHeight - powerBoxHeight - 10; 
                let targetX = houseLeft + 5; // Đặt Tủ điện cách mép trái Ngôi nhà 5px (bên trong)

                // Giới hạn trong màn hình
                targetX = Math.max(0, Math.min(targetX, dropZone.clientWidth - powerBoxWidth));
                targetY = Math.max(0, Math.min(targetY, dropZone.clientHeight - powerBoxHeight));
                
                // Di chuyển Tủ điện
                powerBox.style.left = `${targetX}px`;
                powerBox.style.top = `${targetY}px`;

                // Cập nhật dây dẫn của Tủ điện sau khi nó di chuyển
                if (powerBox.dataset.itemId) {
                    setTimeout(() => {
                         updateConnectedWires(powerBox.dataset.itemId);
                    }, 300); // 300ms khớp với 'transition' trong CSS
                }
            }
        }
        
        /**
         * HÀM SNAP: Tự động gắn Pin mặt trời (solar-panel) vào Ngôi nhà (house)
         */
        function snapSolarPanelToHouse() {
            const house = document.querySelector('#model-a [data-tool="house"]');
            const solarPanel = document.querySelector('#model-a [data-tool="solar-panel"]');

            // Chỉ thực hiện khi cả hai tồn tại
            if (house && solarPanel) {
                const houseTop = parseFloat(house.style.top);
                const houseLeft = parseFloat(house.style.left);
                const houseWidth = parseFloat(house.style.width);
                const houseHeight = parseFloat(house.style.height); // houseHeight is 320px
                
                const solarPanelWidth = parseFloat(solarPanel.style.width); // solarPanel width is 160px
                const solarPanelHeight = parseFloat(solarPanel.style.height); // solarPanel height is 160px

                // Tính toán vị trí mới
                // Đặt vào giữa mái nhà
                // Mái nhà (từ 10% đến 40% chiều cao của SVG)
                let targetX = houseLeft + (houseWidth / 2) - (solarPanelWidth / 2);
                // Đặt nó ở khoảng 15% từ đỉnh của SVG nhà
                let targetY = houseTop + (houseHeight * 0.15) - (solarPanelHeight / 2);

                // Giới hạn trong màn hình
                targetX = Math.max(0, Math.min(targetX, dropZone.clientWidth - solarPanelWidth));
                targetY = Math.max(0, Math.min(targetY, dropZone.clientHeight - solarPanelHeight));
                
                // Di chuyển Pin mặt trời
                solarPanel.style.left = `${targetX}px`;
                solarPanel.style.top = `${targetY}px`;

                // Cập nhật dây dẫn của Pin mặt trời sau khi nó di chuyển
                if (solarPanel.dataset.itemId) {
                    setTimeout(() => {
                         updateConnectedWires(solarPanel.dataset.itemId);
                    }, 300); // 300ms khớp với 'transition'
                }
            }
        }
        
        /**
         * HÀM SNAP: Tự động gắn Công tắc (switch) vào Ngôi nhà (house)
         */
        function snapSwitchToHouse() {
            const house = document.querySelector('#model-a [data-tool="house"]');
            const mainSwitch = document.querySelector('#model-a [data-tool="switch"]');

            if (house && mainSwitch) {
                const houseTop = parseFloat(house.style.top);
                const houseLeft = parseFloat(house.style.left);
                const houseHeight = parseFloat(house.style.height); // 320px
                const houseWidth = parseFloat(house.style.width); // 320px

                const switchWidth = parseFloat(mainSwitch.style.width); // 80px
                const switchHeight = parseFloat(mainSwitch.style.height); // 80px
                
                // Cửa (theo SVG): x="40", y="60", width="20", height="35"
                // (Tỷ lệ 0-100)
                const doorLeftRatio = 40 / 100;
                const doorTopRatio = 60 / 100;
                const doorHeightRatio = 35 / 100;
                const doorWidthRatio = 20 / 100; // Thêm chiều rộng cửa
                
                // Vị trí cửa (đã scale)
                const doorLeftPx = houseLeft + (houseWidth * doorLeftRatio);
                const doorTopPx = houseTop + (houseHeight * doorTopRatio);
                const doorHeightPx = houseHeight * doorHeightRatio;
                const doorWidthPx = houseWidth * doorWidthRatio; // Thêm chiều rộng cửa
                
                // CẬP NHẬT: "giữa mép phải cánh cửa"
                let targetX = doorLeftPx + doorWidthPx + 5; // 5px bên phải cửa
                let targetY = doorTopPx + (doorHeightPx / 2) - (switchHeight / 2); // Giữa cửa (chiều dọc)

                // Giới hạn
                targetX = Math.max(0, Math.min(targetX, dropZone.clientWidth - switchWidth));
                targetY = Math.max(0, Math.min(targetY, dropZone.clientHeight - switchHeight));

                mainSwitch.style.left = `${targetX}px`;
                mainSwitch.style.top = `${targetY}px`;

                if (mainSwitch.dataset.itemId) {
                    setTimeout(() => {
                         updateConnectedWires(mainSwitch.dataset.itemId);
                    }, 300);
                }
            }
        }
        
        /**
         * HÀM SNAP: Tự động gắn Bóng đèn (light-bulb) vào Ngôi nhà (house)
         */
        function snapLightBulbToHouse() {
            const house = document.querySelector('#model-a [data-tool="house"]');
            const lightBulb = document.querySelector('#model-a [data-tool="light-bulb"]');

            if (house && lightBulb) {
                const houseTop = parseFloat(house.style.top);
                const houseLeft = parseFloat(house.style.left);
                const houseHeight = parseFloat(house.style.height); // 320px
                const houseWidth = parseFloat(house.style.width); // 320px

                const bulbWidth = parseFloat(lightBulb.style.width); // 80px
                const bulbHeight = parseFloat(lightBulb.style.height); // 80px
                
                // Cửa (theo SVG): x="40", y="60", width="20"
                const doorLeftRatio = 40 / 100;
                const doorTopRatio = 60 / 100;
                const doorWidthRatio = 20 / 100;
                
                // Vị trí cửa (đã scale)
                const doorLeftPx = houseLeft + (houseWidth * doorLeftRatio);
                const doorTopPx = houseTop + (houseHeight * doorTopRatio);
                const doorWidthPx = houseWidth * doorWidthRatio;

                // "phía trên cánh cửa"
                let targetX = doorLeftPx + (doorWidthPx / 2) - (bulbWidth / 2); // Giữa cửa (chiều ngang)
                let targetY = doorTopPx - bulbHeight - 5; // 5px phía trên cửa

                // Giới hạn
                targetX = Math.max(0, Math.min(targetX, dropZone.clientWidth - bulbWidth));
                targetY = Math.max(0, Math.min(targetY, dropZone.clientHeight - bulbHeight));

                lightBulb.style.left = `${targetX}px`;
                lightBulb.style.top = `${targetY}px`;

                if (lightBulb.dataset.itemId) {
                    setTimeout(() => {
                         updateConnectedWires(lightBulb.dataset.itemId);
                    }, 300);
                }
            }
        }


        // === LOGIC TƯƠNG TÁC CHÍNH ===

        // 1. Nhấn chuột xuống trên HỘP CÔNG CỤ (Màn hình B)
        toolBox.addEventListener('mousedown', (e) => {
            const toolItem = e.target.closest('.tool-item');
            if (!toolItem || toolItem.classList.contains('tool-prohibited')) return; // THÊM: Không cho kéo nếu bị cấm

            e.preventDefault(); 
            isPotentialDrag = true; // Bắt đầu nhấn chuột
            
            // KIỂM TRA CHẾ ĐỘ NỐI DÂY (CLICK)
            if (toolItem.dataset.tool === 'wire') {
                isWiringMode = !isWiringMode; // Chuyển đổi trạng thái
                
                if (isWiringMode) {
                    document.body.classList.add('wiring-mode');
                    wireToolCell.classList.add('active-tool');
                } else {
                    document.body.classList.remove('wiring-mode');
                    wireToolCell.classList.remove('active-tool');
                }
                isPotentialDrag = false; // Đây là 1 cú click, không phải drag
                return; // KHÔNG THỰC HIỆN KÉO THẢ
            }
            
            // TẠO BẢN SAO ĐỂ KÉO (Chỉ khi KHÔNG ở chế độ nối dây)
            if (!isWiringMode) {
                // Logic kiểm tra độc quyền đã được dời lên trên
            
                draggedItemCopy = toolItem.cloneNode(true); 
                draggedItemCopy.classList.add('dragging', 'cloned-item');
                draggedItemCopy.classList.remove('tool-item', 'tool-prohibited'); // Xóa class cấm khỏi bản sao

                const rect = toolItem.getBoundingClientRect();
                offsetX = e.clientX - rect.left;
                offsetY = e.clientY - rect.top;

                draggedItemCopy.style.left = `${e.clientX - offsetX}px`;
                draggedItemCopy.style.top = `${e.clientY - offsetY}px`;

                document.body.appendChild(draggedItemCopy);
            }
        });
        
        // 2. NHẤP CHUỘT (CLICK) TRÊN MÀN HÌNH A (ĐỂ BẬT/TẮT CÔNG TẮC)
        dropZone.addEventListener('click', (e) => {
            // SỬA LỖI CLICK: Chỉ xử lý click nếu KHÔNG phải là đang kéo/vừa kéo xong
            if (isDragging || isWiringMode || isPotentialDrag) {
                // isPotentialDrag kiểm tra nếu mouseup xảy ra ngay sau mousedown (là 1 cú click)
                // nhưng nếu nó là 1 cú kéo (isDragging) thì bỏ qua
                if(isDragging) return;
            }
            
            const targetItem = e.target.closest('.cloned-item');
            if (targetItem && targetItem.dataset.tool === 'switch') {
                // Bật/tắt trạng thái
                targetItem.classList.toggle('switch-on');
                
                // Cập nhật toàn bộ mô phỏng (chủ yếu là đèn)
                updateAllSimulationStates();
            }
        });


        // 3. Nhấn chuột xuống trên MÀN HÌNH A (ĐỂ KÉO HOẶC NỐI DÂY)
        dropZone.addEventListener('mousedown', (e) => {
            isPotentialDrag = true; // Bắt đầu nhấn chuột
            const targetItem = e.target.closest('.cloned-item');

            // TRƯỜNG HỢP 1: ĐANG Ở CHẾ ĐỘ NỐI DÂY
            if (isWiringMode) {
                e.preventDefault();
                isPotentialDrag = false; // Nối dây không phải là kéo
                // Phải có item VÀ item đó là một "đầu nối" hợp lệ
                if (targetItem && wiringParticipants.includes(targetItem.dataset.tool)) {
                    // Bắt đầu vẽ dây
                    wiringStartItem = targetItem;
                    const startPos = getCenter(wiringStartItem); 
                    
                    // Tạo dây tạm thời
                    tempWire = document.createElementNS('http://www.w3.org/2000/svg', 'path');
                    tempWire.setAttribute('id', 'temp-wire');
                    tempWire.setAttribute('d', `M ${startPos.x} ${startPos.y} L ${startPos.x} ${startPos.y}`);
                    wireLayer.appendChild(tempWire);
                }
                return; 
            }
            
            // TRƯỜNG HỢP 2: DI CHUYỂN CÔNG CỤ ĐÃ CÓ
            if (targetItem) {
                // SỬA LỖI CLICK: Chỉ preventDefault NẾU KHÔNG PHẢI LÀ CÔNG TẮC
                // để cho phép sự kiện 'click' hoạt động
                if (targetItem.dataset.tool !== 'switch') {
                    e.preventDefault(); // Ngăn chặn text selection
                }
                
                draggedItem = targetItem;
                // SỬA LỖI CLICK: Đặt z-index ngay lập tức, nhưng chưa add 'dragging'
                // 'dragging' sẽ được thêm trong 'mousemove' nếu chuột thực sự di chuyển
                draggedItem.style.zIndex = 1001; 
                
                const rect = draggedItem.getBoundingClientRect();
                offsetX = e.clientX - rect.left;
                offsetY = e.clientY - rect.top;
            }
        });

        // 4. DI CHUYỂN CHUỘT (Global)
        document.addEventListener('mousemove', (e) => {
            e.preventDefault();
            
            // SỬA LỖI CLICK: Nếu đang nhấn chuột (potentialDrag), và chuột di chuyển -> xác nhận là KÉO
            if (isPotentialDrag && (draggedItem || draggedItemCopy)) {
                // Chỉ đặt isDragging nếu thực sự di chuyển một khoảng nhỏ
                if (Math.abs(e.movementX) > 1 || Math.abs(e.movementY) > 1) {
                     isDragging = true; 
                    // Thêm class dragging khi bắt đầu kéo
                    if(draggedItem && !draggedItem.classList.contains('dragging')) draggedItem.classList.add('dragging');
                    if(draggedItemCopy && !draggedItemCopy.classList.contains('dragging')) draggedItemCopy.classList.add('dragging'); // Thêm cho clone
                }
            }

            const mousePos = getMousePos(e);

            // A. Đang kéo (di chuyển) item
            if (draggedItem && isDragging) {
                let x = mousePos.x - offsetX;
                let y = mousePos.y - offsetY;

                x = Math.max(0, Math.min(x, dropZone.clientWidth - draggedItem.clientWidth));
                y = Math.max(0, Math.min(y, dropZone.clientHeight - draggedItem.clientHeight));
                
                draggedItem.style.left = `${x}px`;
                draggedItem.style.top = `${y}px`;

                // CẬP NHẬT DÂY DẪN (LUÔN LUÔN)
                if (draggedItem.dataset.itemId) {
                    updateConnectedWires(draggedItem.dataset.itemId);
                }
                
                // CẬP NHẬT MÂY CHE/MẶT TRỜI (NẾU ĐANG KÉO MÂY/MẶT TRỜI)
                const toolType = draggedItem.dataset.tool;
                if (toolType === 'cloud' || toolType === 'sun') {
                    updateAllSimulationStates();
                }
            }
            // B. Đang kéo (sao chép) item mới
            else if (draggedItemCopy && isDragging) {
                draggedItemCopy.style.left = `${e.clientX - offsetX}px`;
                draggedItemCopy.style.top = `${e.clientY - offsetY}px`;
            }
            // C. Đang kéo (nối dây)
            else if (wiringStartItem && tempWire) {
                const startPos = getCenter(wiringStartItem);
                // Vẽ đường cong mượt theo vị trí chuột (cho mục đích xem trước)
                tempWire.setAttribute('d', `M ${startPos.x} ${startPos.y} Q ${startPos.x} ${mousePos.y} ${mousePos.x} ${mousePos.y}`);
            }
        });

        // 5. NHẢ CHUỘT (Global)
        document.addEventListener('mouseup', (e) => {
            const dropZoneRect = dropZone.getBoundingClientRect();
            const toolBoxRect = toolBox.getBoundingClientRect();

            const isOverDropZone = e.clientX >= dropZoneRect.left && e.clientX <= dropZoneRect.right &&
                                 e.clientY >= dropZoneRect.top && e.clientY <= dropZoneRect.bottom;
            
            const isOverToolBox = e.clientX >= toolBoxRect.left && e.clientX <= toolBoxRect.right &&
                                e.clientY >= toolBoxRect.top && e.clientY <= toolBoxRect.bottom;

            // TRƯỜNG HỢP 1: KẾT THÚC NỐI DÂY
            if (isWiringMode && wiringStartItem) {
                const endItem = e.target.closest('.cloned-item');
                const startType = wiringStartItem.dataset.tool;
                const endType = endItem ? endItem.dataset.tool : null;

                // Kiểm tra mục tiêu hợp lệ: Phải là đầu nối hợp lệ, không phải chính nó
                if (endItem && endItem !== wiringStartItem && wiringParticipants.includes(endType)) 
                {
                    createPermanentWire(wiringStartItem, endItem);
                    updateAllSimulationStates(); // Cập nhật toàn bộ
                }

                // Reset trạng thái nối dây
                if(tempWire) tempWire.remove();
                tempWire = null;
                wiringStartItem = null;
                
                // TẮT CHẾ ĐỘ NỐI DÂY VÀ HIỂN THỊ
                isWiringMode = false;
                document.body.classList.remove('wiring-mode');
                wireToolCell.classList.remove('active-tool');
                
                // return; // Không return vội, để reset isDragging
            }
            
            // TRƯỜNG HỢP 2: THẢ CÔNG CỤ MỚI (CLONE)
            else if (draggedItemCopy) {
                draggedItemCopy.classList.remove('dragging');
                
                // Thêm kiểm tra
                if (draggedItemCopy.parentNode === document.body) {
                    document.body.removeChild(draggedItemCopy); 
                }
                
                const newToolType = draggedItemCopy.dataset.tool;
                
                if (isOverDropZone && isDragging) { // Chỉ thả nếu thực sự kéo
                    const mousePos = getMousePos(e);
                    
                    let x = mousePos.x - offsetX;
                    let y = mousePos.y - offsetY;

                    // LOGIC PHÓNG TO
                    let scaleFactor = 1;
                    if (newToolType === 'house') {
                        scaleFactor = 4;
                    } else if (newToolType === 'wind-turbine') {
                        scaleFactor = 3;
                    } else if (newToolType === 'solar-panel') { 
                        scaleFactor = 2;
                    }
                    
                    const newWidth = 80 * scaleFactor;
                    const newHeight = 80 * scaleFactor;

                    // Điều chỉnh vị trí thả dựa trên kích thước mới (tránh tràn lề)
                    x = Math.max(0, Math.min(x, dropZone.clientWidth - newWidth));
                    y = Math.max(0, Math.min(y, dropZone.clientHeight - newHeight));
                    
                    draggedItemCopy.style.left = `${x}px`;
                    draggedItemCopy.style.top = `${y}px`;
                    draggedItemCopy.style.width = `${newWidth}px`; 
                    draggedItemCopy.style.height = `${newHeight}px`;
                    
                    // SỬA ĐỔI: Đặt z-index
                    if (newToolType === 'house') {
                        draggedItemCopy.style.zIndex = 5; // Dưới cùng
                    } else if (newToolType === 'solar-panel') {
                        draggedItemCopy.style.zIndex = 8; // Trên nhà, dưới dây
                    } else {
                        draggedItemCopy.style.zIndex = 20; // Mặc định
                    }

                    // Gán ID duy nhất cho item mới (để quản lý dây dẫn sau này)
                    draggedItemCopy.dataset.itemId = generateUUID();
                    
                    dropZone.appendChild(draggedItemCopy);

                    // CẬP NHẬT: Gọi hàm snap Tủ điện/Pin mặt trời
                    updateAllSnapPositions();
                    
                    // CẬP NHẬT TOÀN BỘ MÔ PHỎNG
                    updateAllSimulationStates();
                }
                draggedItemCopy = null;
            }
            
            // TRƯỜNG HỢP 3: THẢ CÔNG CỤ ĐANG DI CHUYỂN (MOVE/DELETE)
            else if (draggedItem) {
                draggedItem.classList.remove('dragging'); // Xóa class dragging
                
                if (isOverToolBox && isDragging) { // Chỉ xóa nếu thực sự kéo
                    const removedItemId = draggedItem.dataset.itemId;
                    
                    // XÓA DÂY DẪN LIÊN QUAN
                    if (removedItemId) {
                        removeWires(removedItemId);
                    }

                    draggedItem.remove(); 
                    
                } else if (isDragging) { // Chỉ thực hiện logic thả nếu thực sự kéo
                    // Reset z-index chính xác khi thả
                    if (draggedItem.dataset.tool === 'house') {
                        draggedItem.style.zIndex = 5; 
                    } else if (draggedItem.dataset.tool === 'solar-panel') {
                        draggedItem.style.zIndex = 8;
                    } else {
                        draggedItem.style.zIndex = 20; 
                    }

                    // Cập nhật dây dẫn lần cuối khi thả
                    if (draggedItem.dataset.itemId) {
                        updateConnectedWires(draggedItem.dataset.itemId);
                    }
                    
                    // CẬP NHẬT: Gọi hàm snap khi di chuyển Ngôi nhà
                    if (draggedItem.dataset.tool === 'house') {
                        updateAllSnapPositions();
                    }
                }
                // Nếu không kéo (isDragging là false), thì đây là một cú click
                // và không cần làm gì ở đây cả (click đã được xử lý)
                
                // CẬP NHẬT TOÀN BỘ MÔ PHỎNG (sau khi xóa hoặc thả)
                updateAllSimulationStates();
                draggedItem = null; 
            }
            
            // SỬA LỖI CLICK: Reset trạng thái kéo
            isDragging = false;
            isPotentialDrag = false;
        });
        
        // KHỞI CHẠY MÔ PHỎNG LẦN ĐẦU
        updateAllSimulationStates();

    </script>
</body>
</html>
