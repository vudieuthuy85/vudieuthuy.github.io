<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Phiếu học tập - Ma trận Sắc màu</title>
    <style>
        /* =========================================================
           CSS RESET & BỐ CỤC 16:9
           ========================================================= */
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        body { 
            background-color: #222; 
            display: flex; 
            align-items: center; 
            justify-content: center; 
            height: 100vh; 
            overflow: hidden; 
        }

        /* Khung 16:9 Full Màn Hình */
        #app {
            background-color: #fff;
            display: flex;
            width: 100vw;
            height: 56.25vw; /* Tỉ lệ 16:9 chuẩn */
            max-height: 100vh;
            max-width: 177.78vh; 
            box-shadow: 0 0 20px rgba(0,0,0,0.8);
            position: relative;
        }

        /* Chia màn hình 50-50 */
        #left-panel {
            width: 50%;
            background-color: #1a1a1a;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 1%;
        }

        #right-panel {
            width: 50%;
            background-color: #f4f6f8;
            display: flex;
            flex-direction: column;
            padding: 1%;
            overflow-y: auto;
        }

        /* Header cho in ấn (Ẩn trên nền tảng Web) */
        #print-header { display: none; }

        /* =========================================================
           BẢNG MA TRẬN SẮC MÀU
           ========================================================= */
        .matrix-row {
            display: flex;
            justify-content: center;
            width: 100%;
        }
        .matrix-char {
            width: 1.4vw;
            height: 1.4vw;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.1vw;
            font-weight: bold;
            font-family: 'Courier New', Courier, monospace;
            border: 1px solid rgba(255, 255, 255, 0.05);
            margin: 1px;
            color: #555;
            transition: all 0.4s ease;
            /* Tự động viết Hoa để che giấu mật mã chữ thường ở dòng cuối */
            text-transform: uppercase; 
        }

        /* CÁC CLASS MÀU SẮC ĐƯỢC CHỈ ĐỊNH */
        .color-VANG { background-color: #FFEB3B !important; color: #000 !important; text-shadow: 0 0 5px #fff; }
        .color-XANHLA { background-color: #4CAF50 !important; color: #fff !important; text-shadow: 0 0 5px #fff; }
        .color-NAU { background-color: #795548 !important; color: #fff !important; text-shadow: 0 0 5px #fff; }
        .color-DO { background-color: #F44336 !important; color: #fff !important; text-shadow: 0 0 5px #fff; }
        .color-CAM { background-color: #FF9800 !important; color: #fff !important; text-shadow: 0 0 5px #fff; }
        .color-CAMDAM { background-color: #E65100 !important; color: #fff !important; text-shadow: 0 0 5px #fff; }
        .color-TIMDAM { background-color: #4A148C !important; color: #fff !important; text-shadow: 0 0 5px #fff; }
        .color-HONGDAM { background-color: #E91E63 !important; color: #fff !important; text-shadow: 0 0 5px #fff; }
        .color-XANHDUONG { background-color: #2196F3 !important; color: #fff !important; text-shadow: 0 0 5px #fff; box-shadow: 0 0 8px #2196F3; }

        /* =========================================================
           KHUNG CÂU HỎI TRẮC NGHIỆM
           ========================================================= */
        #questions-container { flex: 1; overflow-y: auto; padding-right: 8px; }
        .question-block { margin-bottom: 12px; background: #fff; padding: 10px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.08); border-left: 4px solid #3f51b5; }
        .question-title { font-weight: 700; font-size: 1.05vw; margin-bottom: 6px; color: #2c3e50; }
        .options-container { display: flex; flex-direction: column; gap: 4px; }
        .option { 
            padding: 6px 8px; background: #f8f9fa; border-radius: 5px; cursor: pointer; transition: 0.2s; font-size: 0.95vw; border: 1px solid #e0e0e0;
        }
        .option:hover { background: #e3f2fd; border-color: #90caf9; }
        
        /* Trạng thái đúng/sai */
        .option.correct { background: #4CAF50 !important; color: white !important; border-color: #388E3C !important; font-weight: bold; pointer-events: none;}
        .option.wrong { background: #F44336 !important; color: white !important; border-color: #D32F2F !important; pointer-events: none; }
        .question-block.answered-correct .option { pointer-events: none; } 

        /* Nút Điều Khiển */
        .controls { display: flex; flex-wrap: wrap; gap: 8px; margin-top: 10px; padding-top: 10px; border-top: 2px solid #ddd; justify-content: center;}
        .btn { padding: 8px 12px; font-size: 0.9vw; font-weight: bold; border: none; border-radius: 4px; cursor: pointer; color: #fff; transition: 0.2s; }
        #btn-check { background-color: #9C27B0; }
        #btn-retry-wrong { background-color: #FF9800; }
        #btn-retry-all { background-color: #F44336; }
        #btn-pdf { background-color: #2196F3; }
        .btn:hover { filter: brightness(1.1); transform: translateY(-2px); }

        /* Modal Thông điệp hoàn thành */
        #modal-overlay {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85); display: none; align-items: center; justify-content: center; z-index: 1000;
        }
        #modal-content {
            background: #fff; padding: 3vw; border-radius: 12px; text-align: center; max-width: 70%;
            animation: pop 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
            border: 5px solid #FF9800;
        }
        #modal-content h1 { color: #E91E63; font-size: 3vw; margin-bottom: 20px; text-transform: uppercase; line-height: 1.4;}
        #btn-close-modal { padding: 10px 25px; font-size: 1.2vw; background: #333; color: #fff; border: none; border-radius: 5px; cursor: pointer; font-weight: bold;}
        @keyframes pop { 0% { transform: scale(0.5); opacity: 0; } 100% { transform: scale(1); opacity: 1; } }

        /* =========================================================
           XUẤT PDF: 1 TRANG A4 KHỔ NGANG (Chứa đủ 14 Câu)
           ========================================================= */
        @media print {
            @page { size: A4 landscape; margin: 8mm; }
            body { height: auto; overflow: visible; background: white; display: block; }
            #app { height: auto; max-height: none; width: 100%; max-width: none; box-shadow: none; display: flex; flex-direction: row; align-items: flex-start; }
            
            /* Nửa trái 40% */
            #left-panel { width: 40%; padding: 0; background: white; color: black; display: block; }
            
            #print-header { display: block; margin-bottom: 12px; font-family: 'Times New Roman', Times, serif; }
            #print-header h2 { text-transform: uppercase; text-align: center; font-size: 15px; margin-bottom: 10px;}
            #print-header p { font-size: 12px; margin-bottom: 6px; font-weight: bold; line-height: 1.2;}
            
            .matrix-char { width: 11px; height: 11px; font-size: 9px; border: 1px solid #ccc; color: #ccc; }
            
            /* Nửa phải 60% (Chia 2 cột ép vừa 14 câu) */
            #right-panel { width: 60%; padding: 0 0 0 12px; background: white; overflow: visible; display: block; }
            #questions-container { overflow: visible; column-count: 2; column-gap: 12px; }
            
            .question-block { margin-bottom: 5px; padding: 0; border: none; box-shadow: none; page-break-inside: avoid; }
            .question-title { font-size: 9px; margin-bottom: 2px; font-family: 'Times New Roman', Times, serif; color: #000; }
            .options-container { gap: 1px; }
            .option { font-size: 8.5px; padding: 1px 2px; border: none; background: transparent; font-family: 'Times New Roman', Times, serif; color: #000;}
            
            .controls, #modal-overlay { display: none !important; }
        }
    </style>
</head>
<body>

<div id="app">
    <div id="left-panel">
        <div id="print-header">
            <h2>Phiếu học tập MA TRẬN SẮC MÀU</h2>
            <p>Họ và tên học sinh:....................................................</p>
            <p>Lớp: ...............................................................................</p>
            <p>Ngày ...........tháng................năm...................</p>
        </div>
        <div id="matrix-container"></div>
    </div>

    <div id="right-panel">
        <div id="questions-container"></div>
        <div class="controls">
            <button id="btn-check" class="btn">Kiểm tra thông điệp</button>
            <button id="btn-retry-wrong" class="btn">Làm lại câu sai</button>
            <button id="btn-retry-all" class="btn">Làm lại tất cả</button>
            <button id="btn-pdf" class="btn">Xuất file PDF</button>
        </div>
    </div>

    <div id="modal-overlay">
        <div id="modal-content">
            <h1>NHÀ BÁC HỌC MICHAEL FARADAY</h1>
            <button id="btn-close-modal">Đóng</button>
        </div>
    </div>
</div>

<script>
// Code Javascript An Toàn Không Sử Dụng Onclick
window.addEventListener("DOMContentLoaded", function () {
    
    // 1. DỮ LIỆU MA TRẬN ĐÃ NÂNG CẤP (Chữ N NÉT CĂNG CHUẨN MẪU)
    // Cụm 'michael faraday' dòng cuối là chữ viết thường để tránh tô màu nhầm lẫn với ký tự Y nền.
    const matrixRaw = [
        "XXYXĐĐXYXIIIYXỆỆỆYXNXXNYZZXXXX",
        "XXYĐĐĐĐXXXIXYXỆXXXYNNXNYZXXZXX",
        "ZXYĐXĐXYXXIXXYỆỆXYXNXNNZYXZXXX",
        "YXXXĐĐXXYIIIXXỆỆỆYXNXXNZZXYXXX",
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
        "XYZTTTXRRXYƯXƯXỜỜỜYNXXNXGGGZXY",
        "XYXXTXYRXRXƯXƯYỜXỜXNNXNYGXXXZY",
        "YXZXTXXRRXYƯXƯXỜXỜYNXNNXGXGYXZ",
        "ZXYXTXYRXRXƯƯƯYỜỜỜXNXXNYGGGXZX",
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
        "XYZXYZXYXĐĐXXYỀỀỀXZUXUZXYXZYXZ",
        "YXZYXZYXĐĐĐĐXZỀXXYXUXUXYZXYZXY",
        "ZXYXZYXZĐXĐXXYỀỀXZXUXUYXZYXZYX",
        "XYZXYZXYXĐĐXXZỀỀỀXYUUUZXYXZYXZ",
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
        "XXXX+|-------->---------|-XXXX",
        "XXXXX|XXXXXXXXXXXXXXXXXX|XXXXX",
        "XXXX+|-------->---------|-XXXX",
        "XXXXX|XXXXXXXXXXXXXXXXXX|XXXXX",
        "XXXX+|-------->---------|-XXXX",
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
        "XXXXXXXmichaelXfaradayXXXXXXXX" 
    ];

    // 2. DỮ LIỆU CÂU HỎI VÀ ÁNH XẠ
    const questionsData = [
        {
            q: "Câu 1: Tính chất cơ bản nhất dùng để nhận biết sự tồn tại của điện trường là gì?",
            opts: ["A. Khả năng tác dụng lực từ lên một nam châm thử đặt trong đó.", "B. Đặc trưng cơ bản của điện trường là tác dụng lực điện lên điện tích đặt trong nó.", "C. Sinh ra nhiệt lượng khi có vật thể đặt gần nó.", "D. Làm phát sáng không khí xung quanh mọi lúc."],
            ansIndex: 1, targetChars: ['Đ'], colorClass: 'color-VANG'
        },
        {
            q: "Câu 2: Hạt mang điện nào sau đây khi đặt trong điện trường sẽ chịu lực điện cùng chiều với vectơ cường độ điện trường?",
            opts: ["A. Electron tự do.", "B. Nơtron (neutron).", "C. Ion dương và hạt proton.", "D. Ion âm."],
            ansIndex: 2, targetChars: ['I'], colorClass: 'color-XANHLA'
        },
        {
            q: "Câu 3: Đặc điểm lực điện tác dụng lên hạt electron khi nó bay dọc theo đường sức điện trường là gì?",
            opts: ["A. Hạt chịu lực tác dụng luôn vuông góc với các đường sức.", "B. Không chịu bất kì lực tác dụng nào vì khối lượng quá nhỏ.", "C. Electron và các ion âm luôn chịu lực tác dụng ngược chiều đường sức điện.", "D. Chịu lực tác dụng hướng thẳng đứng xuống đất do trọng lực lấn át."],
            ansIndex: 2, targetChars: ['Ệ'], colorClass: 'color-NAU'
        },
        {
            q: "Câu 4: Bỏ qua khối lượng của electron, nếu một electron trong điện trường đều có vận tốc ban đầu bằng không thì dưới tác dụng của lực điện trường nó sẽ chuyển động",
            opts: ["A. dọc theo chiều của đường sức điện trường.", "B. vuông góc với đường sức điện trường.", "C. theo một quỹ đạo bất kỳ.", "D. ngược chiều đường sức điện trường."],
            ansIndex: 3, targetChars: ['N'], colorClass: 'color-DO'
        },
        {
            q: "Câu 5: Khi một điện tích chuyển động vào điện trường đều theo phương vuông góc với đường sức điện thì điện trường sẽ không ảnh hưởng tới",
            opts: ["A. gia tốc của chuyển động.", "B. thành phần vận tốc theo phương vuông góc với đường sức điện.", "C. thành phần vận tốc theo phương song song với đường sức điện.", "D. quỹ đạo của chuyển động."],
            ansIndex: 1, targetChars: ['T'], colorClass: 'color-CAM'
        },
        {
            q: "Câu 6: Đặc điểm về hướng của cường độ điện trường sinh bởi một điện tích điểm dương cô lập là",
            opts: ["A. Hướng xoay tròn quanh tâm điện tích.", "B. Ra xa khỏi điện tích gây ra điện trường.", "C. Hướng về phía điện tích gây ra điện trường.", "D. Luôn hướng từ trên xuống dưới so với mặt đất."],
            ansIndex: 1, targetChars: ['R'], colorClass: 'color-CAMDAM'
        },
        {
            q: "Câu 7: Đặc điểm của vectơ cường độ điện trường tại mọi điểm trong điện trường đều là",
            opts: ["A. Độ lớn giảm dần đều theo khoảng cách đến nguồn.", "B. Hướng thay đổi liên tục nhưng độ lớn cố định.", "C. Chỉ có hướng giống nhau, còn độ lớn phụ thuộc vào môi trường.", "D. Giữ nguyên không đổi về cả phương, chiều và độ lớn."],
            ansIndex: 3, targetChars: ['G'], colorClass: 'color-TIMDAM'
        },
        {
            q: "Câu 8: Biểu thức liên hệ giữa hiệu điện thế (U) và cường độ điện trường (E) trong khoảng không gian giữa hai bản phẳng song song là gì?",
            opts: ["A. U = E + d", "B. E = U.d", "C. U = E.d (với d là khoảng cách giữa hai bản).", "D. d = E.U"],
            ansIndex: 2, targetChars: ['U'], colorClass: 'color-XANHDUONG'
        },
        {
            q: "Câu 9: Vị trí nào đối với một vật dẫn bằng kim loại đang ở trạng thái cân bằng tĩnh điện sẽ KHÔNG có điện trường (cường độ bằng 0)?",
            opts: ["A. Ngay sát bề mặt bên ngoài của vật dẫn.", "B. Ở (tâm) của quả cầu kim loại và toàn bộ phần thể tích rỗng bên trong nó.", "C. Chỉ duy nhất ở vô cực.", "D. Tại các mũi nhọn của vật dẫn."],
            ansIndex: 1, targetChars: ['Ờ'], colorClass: 'color-HONGDAM'
        },
        {
            q: "Câu 10: Một điện tích q = 2 μC chịu tác dụng của lực điện F = 0,01 N khi đặt tại một điểm M. Cường độ điện trường tại M là bao nhiêu?",
            opts: ["A. 5000 V/m", "B. 50 V/m", "C. 0,5 V/m", "D. 20000 V/m"],
            ansIndex: 0, targetChars: ['Ề'], colorClass: 'color-HONGDAM'
        },
        {
            q: "Câu 11: Tính cường độ điện trường do một điện tích điểm Q = 1 nC gây ra tại một điểm cách nó 3 cm trong chân không.",
            opts: ["A. 3.000 V/m", "B. 90.000 V/m", "C. 10.000 V/m", "D. 100 V/m"],
            ansIndex: 2, targetChars: ['Ư'], colorClass: 'color-HONGDAM'
        },
        {
            q: "Câu 12: Hai bản kim loại phẳng đặt song song cách nhau 2 cm. Cường độ điện trường đều giữa hai bản là 300 V/m. Hiệu điện thế giữa hai bản là:",
            opts: ["A. 600 V", "B. 6 V", "C. 150 V", "D. 0,015 V"],
            ansIndex: 1, targetChars: ['-', '>'], colorClass: 'color-CAM'
        },
        {
            q: "Câu 13: Một điện tích q = 1,6.10^-19 C được đặt vào điện trường đều E = 1,25.10^19 V/m. Lực điện tác dụng lên nó có độ lớn là?",
            opts: ["A. 0,2 N", "B. 1,25 N", "C. 2 N", "D. 20 N"],
            ansIndex: 2, targetChars: ['+'], colorClass: 'color-DO'
        },
        {
            q: "Câu 14: Đơn vị của cường độ điện trường trong hệ đo lường quốc tế (SI) là",
            opts: ["A. Vôn trên mét (V/m)", "B. Culông trên mét vuông (C/m²)", "C. Niutơn trên mét (N/m)", "D. Jun (J)"],
            // CHỈ GỌI chính xác 'I' và các ký tự viết thường (để không gọi nhầm chữ Y nền)
            ansIndex: 0, targetChars: ['I', 'm', 'i', 'c', 'h', 'a', 'e', 'l', 'f', 'r', 'd', 'y'], colorClass: 'color-XANHDUONG'
        }
    ];

    // 3. KHỞI TẠO ÂM THANH
    const audioDung = new Audio('dung.mp3');
    const audioSai = new Audio('sai.mp3');

    // 4. RENDER BẢNG MA TRẬN BẰNG DOM API
    const matrixContainer = document.getElementById("matrix-container");
    matrixRaw.forEach(rowText => {
        const rowDiv = document.createElement("div");
        rowDiv.className = "matrix-row";
        for (let char of rowText) {
            const span = document.createElement("span");
            span.className = "matrix-char";
            span.textContent = char;
            span.dataset.char = char; // Thuộc tính Data giúp Javascript nhận diện chính xác kể cả phân biệt Hoa/Thường
            rowDiv.appendChild(span);
        }
        matrixContainer.appendChild(rowDiv);
    });

    // 5. RENDER CÂU HỎI VÀ GẮN EVENTLISTENER
    const quizContainer = document.getElementById("questions-container");

    questionsData.forEach((qObj, qIndex) => {
        const qBlock = document.createElement("div");
        qBlock.className = "question-block";
        qBlock.id = "question-" + qIndex;

        const qTitle = document.createElement("div");
        qTitle.className = "question-title";
        qTitle.textContent = qObj.q;
        qBlock.appendChild(qTitle);

        const optsContainer = document.createElement("div");
        optsContainer.className = "options-container";

        qObj.opts.forEach((optText, optIndex) => {
            const optDiv = document.createElement("div");
            optDiv.className = "option";
            optDiv.textContent = optText;
            
            // Xử lý logic click
            optDiv.addEventListener("click", function () {
                if (qBlock.classList.contains("answered-correct")) return; // Ngăn chặn tương tác nếu câu đã đúng
                
                // Xóa trạng thái sai của đáp án khác
                Array.from(optsContainer.children).forEach(child => child.classList.remove("wrong"));

                if (optIndex === qObj.ansIndex) {
                    optDiv.classList.add("correct");
                    qBlock.classList.add("answered-correct");
                    
                    audioDung.currentTime = 0;
                    audioDung.play().catch(e => console.log("Cần nhấp chuột để trình duyệt cho phép phát âm thanh."));

                    // Logic tô màu
                    if(qObj.targetChars.length > 0) {
                        qObj.targetChars.forEach(target => {
                            const cells = document.querySelectorAll(`.matrix-char[data-char="${target}"]`);
                            cells.forEach(cell => {
                                // Câu 14 sẽ ghi đè class color-XANHDUONG lên lưới
                                cell.className = "matrix-char " + qObj.colorClass; 
                            });
                        });
                    }
                } else {
                    optDiv.classList.add("wrong");
                    audioSai.currentTime = 0;
                    audioSai.play().catch(e => console.log("Cần nhấp chuột để trình duyệt cho phép phát âm thanh."));
                }
            });
            optsContainer.appendChild(optDiv);
        });

        qBlock.appendChild(optsContainer);
        quizContainer.appendChild(qBlock);
    });

    // 6. XỬ LÝ CÁC NÚT TÍNH NĂNG CHUNG
    document.getElementById("btn-check").addEventListener("click", function() {
        const correctCount = document.querySelectorAll('.answered-correct').length;
        if (correctCount === questionsData.length) {
            document.getElementById("modal-overlay").style.display = "flex";
        } else {
            alert("Các em cần hoàn thành đúng toàn bộ 14 câu hỏi để giải mã thông điệp nhé!");
        }
    });

    document.getElementById("btn-close-modal").addEventListener("click", function() {
        document.getElementById("modal-overlay").style.display = "none";
    });

    document.getElementById("btn-retry-wrong").addEventListener("click", function() {
        document.querySelectorAll('.option.wrong').forEach(opt => opt.classList.remove("wrong"));
    });

    document.getElementById("btn-retry-all").addEventListener("click", function() {
        if(confirm("Bạn có chắc chắn muốn xóa toàn bộ bài làm để bắt đầu lại từ đầu?")) {
            document.querySelectorAll('.question-block').forEach(qb => qb.classList.remove("answered-correct"));
            document.querySelectorAll('.option').forEach(opt => {
                opt.classList.remove("correct");
                opt.classList.remove("wrong");
            });
            document.querySelectorAll('.matrix-char').forEach(cell => {
                cell.className = "matrix-char"; // Reset ma trận về màu gốc
            });
        }
    });

    // Tính năng xuất PDF thông minh
    document.getElementById("btn-pdf").addEventListener("click", function() {
        window.print();
    });
});
</script>

</body>
</html>
