# content-calendar
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Content Planner - Auto Sync Stock</title>
    
    <script src="https://cdn.jsdelivr.net/npm/fullcalendar@6.1.10/index.global.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;600;700&display=swap" rel="stylesheet">
    
    <style>
        body { font-family: 'Sarabun', sans-serif; background-color: #f1f5f9; color: #1e293b; }
        .fc-event { cursor: grab; padding: 5px 8px; border-radius: 6px; border: none !important; font-size: 0.75rem; color: white !important; }
        #modalOverlay { display: none; backdrop-filter: blur(8px); z-index: 999; }
        
        #external-events { max-height: 450px; overflow-y: auto; padding-right: 4px; }
        #external-events::-webkit-scrollbar { width: 4px; }
        #external-events::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }

        .fc-event-external { 
            cursor: grab; margin-bottom: 10px; padding: 14px; border-radius: 14px; 
            color: white; font-weight: bold; font-size: 0.8rem; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
            transition: all 0.2s ease; border: 2px solid transparent;
        }
        .fc-event-external:hover { transform: translateY(-2px); border-color: rgba(255,255,255,0.3); }
        .sidebar-fixed { height: calc(100vh - 48px); position: sticky; top: 24px; }
    </style>
</head>
<body class="p-4 lg:p-6">

    <div class="max-w-[1600px] mx-auto">
        <div class="flex flex-col lg:flex-row gap-6">
            
            <aside class="w-full lg:w-80 shrink-0">
                <div class="bg-white p-6 rounded-3xl shadow-sm border border-slate-200 sidebar-fixed flex flex-col">
                    
                    <div class="mb-6">
                        <h2 class="text-xl font-bold mb-6 flex items-center gap-2">
                            <span class="w-2 h-6 bg-indigo-600 rounded-full"></span>
                            เพิ่มงานใหม่
                        </h2>
                        <div class="space-y-4">
                            <input type="text" id="contentTitle" placeholder="ชื่อหัวข้อคอนเทนต์..." 
                                class="w-full bg-slate-50 border border-slate-100 p-3 rounded-2xl focus:ring-2 focus:ring-indigo-500 outline-none transition-all">
                            
                            <div class="grid grid-cols-2 gap-2">
                                <div>
                                    <label class="block text-[10px] font-black text-slate-400 uppercase mb-1 ml-1 tracking-widest">ช่องทาง</label>
                                    <select id="contentPlatform" class="w-full bg-slate-50 border border-slate-100 p-2.5 rounded-xl outline-none text-xs font-bold">
                                        <option value="#1877F2">FB (Blue)</option>
                                        <option value="#000000">TikTok (Black)</option>
                                    </select>
                                </div>
                                <div>
                                    <label class="block text-[10px] font-black text-slate-400 uppercase mb-1 ml-1 tracking-widest">หมวดหมู่</label>
                                    <select id="contentCategory" class="w-full bg-slate-50 border border-slate-100 p-2.5 rounded-xl outline-none text-xs font-bold">
                                        <option value="INFO">INFO</option>
                                        <option value="PROD">PROD</option>
                                        <option value="LIFE">LIFE</option>
                                        <option value="SERV">SERV</option>
                                    </select>
                                </div>
                            </div>

                            <button onclick="addToStock()" class="w-full bg-indigo-600 text-white font-bold py-3.5 rounded-2xl hover:bg-indigo-700 transition-all active:scale-95 shadow-lg shadow-indigo-100">
                                เก็บเข้าคลังสต็อก
                            </button>
                        </div>
                    </div>

                    <hr class="border-slate-100 mb-6">

                    <div class="flex-grow flex flex-col min-height-0 overflow-hidden">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-xs font-black text-slate-400 uppercase tracking-widest">คลังคอนเทนต์</h3>
                            <span id="stockCount" class="bg-slate-100 text-slate-500 text-[10px] px-2 py-0.5 rounded-full font-bold">0</span>
                        </div>
                        <div id="external-events" class="flex-grow space-y-1">
                            </div>
                        <p class="text-[10px] text-slate-400 mt-4 italic text-center">ลากงานไปวางในปฏิทินเพื่อจองวัน</p>
                    </div>
                </div>
            </aside>

            <main class="flex-grow">
                <div class="bg-white p-6 rounded-3xl shadow-sm border border-slate-200">
                    <div id="calendar"></div>
                </div>
            </main>

        </div>
    </div>

    <div id="modalOverlay" class="fixed inset-0 flex items-center justify-center p-4">
        <div class="bg-white rounded-[2.5rem] max-w-lg w-full p-8 shadow-2xl relative border border-slate-100 animate-in fade-in zoom-in duration-200">
            <div id="modalColorStrip" class="absolute top-0 left-0 right-0 h-4 rounded-t-[2.5rem]"></div>
            <button onclick="closeModal()" class="absolute top-6 right-6 w-10 h-10 flex items-center justify-center bg-slate-50 rounded-full text-slate-400 hover:text-rose-500 hover:bg-rose-50 transition-all text-2xl font-bold shadow-sm">&times;</button>
            
            <div class="mt-4">
                <div class="flex gap-2 mb-4">
                    <span id="modalPlatformLabel" class="text-[10px] font-black text-white px-3 py-1 rounded-full uppercase tracking-widest shadow-sm">PLATFORM</span>
                    <span id="modalCatLabel" class="text-[10px] font-black border border-slate-200 text-slate-400 px-3 py-1 rounded-full uppercase tracking-widest">CAT</span>
                </div>

                <input type="text" id="viewTitle" class="text-2xl font-bold text-slate-800 leading-tight mb-6 w-full bg-transparent border-b border-transparent focus:border-slate-200 outline-none" placeholder="ระบุหัวข้อ...">
                
                <div class="max-h-[45vh] overflow-y-auto pr-2">
                    <div class="mb-6">
                        <label class="text-[10px] font-bold text-slate-400 uppercase mb-2 block tracking-widest">แคปชั่น / รายละเอียด</label>
                        <textarea id="viewCaption" rows="6" class="w-full bg-slate-50 p-5 rounded-3xl text-slate-700 text-sm border border-slate-100 focus:bg-white focus:ring-2 focus:ring-indigo-500 outline-none transition-all" placeholder="ยังไม่มีรายละเอียด..."></textarea>
                    </div>

                    <div id="downloadSection" class="mb-2">
                        <label class="text-[10px] font-bold text-slate-400 uppercase mb-2 block tracking-widest">ลิงก์แนบไฟล์งาน</label>
                        <div class="flex flex-col gap-2">
                            <input type="text" id="viewLink" class="w-full bg-slate-50 p-3 rounded-xl text-xs text-blue-600 border border-slate-100 outline-none" placeholder="แปะลิงก์ที่นี่...">
                            <a id="downloadBtn" href="#" target="_blank" class="w-full bg-emerald-500 text-white p-4 rounded-2xl font-bold hover:bg-emerald-600 transition-all flex items-center justify-center gap-2 shadow-lg shadow-emerald-100">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a2 2 0 002 2h10a2 2 0 002-2v-1m-4-4l-4 4m0 0l-4-4m4 4V4" /></svg>
                                ดาวน์โหลดไฟล์
                            </a>
                        </div>
                    </div>
                </div>

                <div class="flex gap-3 mt-8 pt-6 border-t border-slate-50">
                    <button onclick="deleteEvent()" class="flex-1 bg-rose-50 text-rose-500 py-3.5 rounded-2xl font-bold hover:bg-rose-100 transition-all text-sm">ลบงาน</button>
                    <button onclick="saveChanges()" class="flex-[2] bg-slate-900 text-white py-3.5 rounded-2xl font-bold hover:bg-slate-800 shadow-xl transition-all text-sm">บันทึกข้อมูล</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        let calendar;
        let currentEvent = null;

        document.addEventListener('DOMContentLoaded', function() {
            const containerEl = document.getElementById('external-events');
            
            // โหลดข้อมูล
            const savedCal = JSON.parse(localStorage.getItem('calData_final')) || [];
            const savedStock = JSON.parse(localStorage.getItem('stockData_final')) || [];

            // ตั้งค่า Calendar
            calendar = new FullCalendar.Calendar(document.getElementById('calendar'), {
                initialView: 'dayGridMonth',
                locale: 'th',
                editable: true,
                droppable: true,
                events: savedCal,
                eventClick: (info) => openModal(info.event),
                
                // --- จุดสำคัญ: เมื่อรับงานจากคลัง ---
                eventReceive: function(info) {
                    // 1. เก็บข้อมูลเบื้องหลัง
                    info.event.setExtendedProp('category', info.draggedEl.dataset.category);
                    info.event.setExtendedProp('caption', info.draggedEl.dataset.caption);
                    info.event.setExtendedProp('link', info.draggedEl.dataset.link);
                    
                    // 2. ลบออกจากหน้าจอคลังทันที
                    info.draggedEl.remove();
                    
                    // 3. บันทึกข้อมูลลง LocalStorage (ทั้งปฏิทินและคลังที่ถูกลบไปแล้ว)
                    sync(); 
                },
                
                eventContent: function(arg) {
                    const cat = arg.event.extendedProps.category || "N/A";
                    return {
                        html: `<div class="p-1 truncate font-bold text-[10px]">
                                <span class="opacity-60 font-black">[${cat}]</span> ${arg.event.title}
                               </div>`
                    }
                },
                eventDrop: () => sync(),
                eventResize: () => sync()
            });
            calendar.render();

            // ตั้งค่าระบบ Drag
            new FullCalendar.Draggable(containerEl, {
                itemSelector: '.fc-event-external',
                eventData: function(el) {
                    return {
                        title: el.dataset.titleOnly,
                        backgroundColor: el.style.backgroundColor,
                        extendedProps: { 
                            category: el.dataset.category,
                            caption: el.dataset.caption,
                            link: el.dataset.link 
                        }
                    };
                }
            });

            // แสดงผลคลังจาก Memory
            savedStock.forEach(s => createStockItem(s.title, s.platformColor, s.category, s.caption, s.link));
            updateStockCount();
        });

        function addToStock() {
            const title = document.getElementById('contentTitle').value;
            const platformColor = document.getElementById('contentPlatform').value;
            const category = document.getElementById('contentCategory').value;
            
            if (!title) return alert("กรุณาใส่หัวข้อคอนเทนต์");

            createStockItem(title, platformColor, category, "", "");
            sync(); // บันทึกลง Storage
            document.getElementById('contentTitle').value = "";
        }

        function createStockItem(title, color, category, caption, link) {
            const container = document.getElementById('external-events');
            const el = document.createElement('div');
            el.className = 'fc-event-external';
            el.style.backgroundColor = color;
            el.dataset.titleOnly = title;
            el.dataset.category = category;
            el.dataset.caption = caption;
            el.dataset.link = link;
            el.innerText = `[${category}] ${title}`;
            container.appendChild(el);
            updateStockCount();
        }

        function openModal(event) {
            currentEvent = event;
            const props = event.extendedProps;
            document.getElementById('viewTitle').value = event.title;
            document.getElementById('viewCaption').value = props.caption || "";
            document.getElementById('viewLink').value = props.link || "";
            
            const platLabel = document.getElementById('modalPlatformLabel');
            platLabel.style.backgroundColor = event.backgroundColor;
            platLabel.innerText = event.backgroundColor === '#1877F2' ? 'FACEBOOK' : 'TIKTOK';
            
            document.getElementById('modalCatLabel').innerText = props.category || "General";
            document.getElementById('modalColorStrip').style.backgroundColor = event.backgroundColor;
            document.getElementById('downloadBtn').href = props.link || "#";
            document.getElementById('modalOverlay').style.display = 'flex';
        }

        function saveChanges() {
            currentEvent.setProp('title', document.getElementById('viewTitle').value);
            currentEvent.setExtendedProp('caption', document.getElementById('viewCaption').value);
            currentEvent.setExtendedProp('link', document.getElementById('viewLink').value);
            sync();
            closeModal();
        }

        function closeModal() { document.getElementById('modalOverlay').style.display = 'none'; }
        function deleteEvent() { if(confirm("ลบงานนี้ใช่ไหม?")) { currentEvent.remove(); sync(); closeModal(); } }
        function updateStockCount() { document.getElementById('stockCount').innerText = document.querySelectorAll('.fc-event-external').length; }

        // --- ฟังก์ชัน Sync หัวใจหลัก ---
        function sync() {
            // บันทึกรายการบนปฏิทิน
            const calData = calendar.getEvents().map(e => ({
                title: e.title, start: e.startStr, backgroundColor: e.backgroundColor, extendedProps: e.extendedProps
            }));
            localStorage.setItem('calData_final', JSON.stringify(calData));

            // บันทึกรายการในคลัง (ดึงเฉพาะที่ยังเหลืออยู่ใน DOM)
            const stockData = Array.from(document.querySelectorAll('.fc-event-external')).map(el => ({
                title: el.dataset.titleOnly, platformColor: el.style.backgroundColor, category: el.dataset.category, caption: el.dataset.caption, link: el.dataset.link
            }));
            localStorage.setItem('stockData_final', JSON.stringify(stockData));
            updateStockCount();
        }
    </script>
</body>
</html>
