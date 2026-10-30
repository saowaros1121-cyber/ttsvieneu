# VieNeu-TTS Desktop & Web — Tạo giọng nói tiếng Việt trên máy tính

Ứng dụng **Text-to-Speech (TTS) và Voice Cloning chạy local**, phát triển trên mã nguồn mở [VieNeu-TTS](https://github.com/pnnbao97/VieNeu-TTS). Bản này bổ sung giao diện **Tkinter cho Windows**, xử lý nhiều kịch bản TXT theo hàng đợi, tách câu thông minh, thiết lập thời gian ngắt nghỉ, lựa chọn thư mục xuất và chế độ kiểm tra âm thanh **Quality+**.

> **Bắt đầu nhanh:** Trên Windows, giải nén toàn bộ mã nguồn vào một thư mục rồi nhấp đúp **`run_tkinter.bat`**. Ở lần chạy đầu, trình khởi động sẽ chuẩn bị Python 3.12, môi trường `.venv` và các thư viện cần thiết. Cần kết nối Internet để tải phần mềm và model chưa có trên máy.

## Mục lục

- [Hai cách sử dụng](#hai-cách-sử-dụng)
- [Cài đặt và khởi chạy trên Windows](#cài-đặt-và-khởi-chạy-trên-windows)
- [Hướng dẫn sử dụng Tkinter](#hướng-dẫn-sử-dụng-tkinter)
- [Xử lý nhiều file TXT](#xử-lý-nhiều-file-txt)
- [Tách câu thông minh và ngắt nghỉ](#tách-câu-thông-minh-và-ngắt-nghỉ)
- [Chế độ Quality+](#chế-độ-quality)
- [Cài đặt mặc định](#cài-đặt-mặc-định)
- [Xử lý lỗi thường gặp](#xử-lý-lỗi-thường-gặp)
- [Cấu trúc dự án và tài liệu](#cấu-trúc-dự-án-và-tài-liệu)
- [Nguồn gốc và giấy phép](#nguồn-gốc-và-giấy-phép)

## Hai cách sử dụng

| Giao diện | Cách mở | Phù hợp khi |
| --- | --- | --- |
| **Tkinter Desktop** | `run_tkinter.bat` | Muốn dùng ứng dụng Windows riêng, không cần trình duyệt, quản lý hàng đợi TXT và thư mục xuất. |
| **Gradio Web** | `run.bat` sau khi đã chuẩn bị `.venv` | Muốn dùng giao diện web VieNeu-TTS tại `http://localhost:7860`. |

Tkinter sử dụng trực tiếp phần lõi VieNeu-TTS trong dự án, không yêu cầu bật máy chủ Gradio. **Không nên hiểu hai giao diện đã có mọi nút và mọi tính năng giống hệt nhau**; xem chức năng desktop bên dưới và [hướng dẫn Tkinter](docs/tkinter-app.md).

## Cài đặt và khởi chạy trên Windows

### Cách 1 — Tự động (khuyến nghị)

1. Tải mã nguồn từ GitHub bằng **Code → Download ZIP**, hoặc lấy ZIP dự án đã đóng gói.
2. **Giải nén toàn bộ ZIP**; không chạy trực tiếp file `.bat` từ bên trong ZIP.
3. Mở thư mục dự án, nhấp đúp **`run_tkinter.bat`**.
4. Trình chạy tự tìm `uv` (hoặc cài `uv` từ Astral), tạo `.venv` bằng **Python 3.12**, cài thư viện phù hợp **CPU / NVIDIA GPU**, kiểm tra môi trường rồi mở ứng dụng.
5. Trong cửa sổ Tkinter, chọn model, thiết bị và bấm **Tải model** trước khi tạo audio.

Lần đầu cần Internet và dung lượng trống để tải thư viện/model; những lần sau tái sử dụng `.venv`. Nếu nhận diện được NVIDIA, script thử cài CUDA; trường hợp cài CUDA lỗi trong chế độ tự động, script thử chuyển sang CPU. Máy không có NVIDIA vẫn có thể chạy qua CPU, nhưng tốc độ tùy cấu hình.

Muốn chỉ định chế độ cài đặt, mở **Command Prompt** tại thư mục dự án và chạy:

```bat
set VIENEU_INSTALL_MODE=cpu
run_tkinter.bat
```

Hoặc dùng `set VIENEU_INSTALL_MODE=gpu` nếu đã có GPU NVIDIA/CUDA phù hợp. Biến này chỉ áp dụng cho cửa sổ Command Prompt hiện tại. **Chế độ ép `gpu` sẽ báo lỗi nếu cài CUDA không thành công**, không tự chuyển sang CPU.

### Cách 2 — Cài thủ công cho người phát triển

Nếu đã cài [`uv`](https://docs.astral.sh/uv/), tại thư mục dự án chạy:

```powershell
# CPU, kèm thư viện đọc PDF
uv sync --python 3.12 --extra pdf

# Hoặc GPU NVIDIA (thay cho lệnh CPU bên trên)
uv sync --python 3.12 --extra pdf --extra cuda

# Mở desktop
uv run python -m apps.tkinter_app
```

### Chạy giao diện web

Sau khi có môi trường `.venv`, nhấp đúp **`run.bat`** để mở Gradio trên **`http://localhost:7860`**. File này **không tự cài thư viện** và hiện có thao tác **đóng tiến trình đang chiếm cổng 7860** trước khi khởi động; hãy lưu công việc nếu đang chạy ứng dụng khác trên cổng đó.

Nếu muốn tự khởi động web mà không dùng `run.bat`, tại thư mục dự án chạy:

```powershell
uv run python -m apps.gradio_main
```

## Hướng dẫn sử dụng Tkinter

Ứng dụng có năm tab chính:

| Tab | Chức năng |
| --- | --- |
| **Đọc truyện / Clone** | Đọc bằng giọng có sẵn (Preset) hoặc clone từ audio mẫu; nhập TXT/PDF; tách câu, chỉnh ngắt nghỉ, tạo WAV/MP3, nghe kết quả. |
| **TXT hàng loạt** | Thêm nhiều file TXT, thay đổi thứ tự, tự tách câu từng file rồi xử lý lần lượt, chọn thư mục xuất. |
| **Hội thoại** | Kịch bản nhiều nhân vật, mỗi dòng theo mẫu `Tên: Lời thoại`; ánh xạ nhân vật sang giọng Preset. |
| **Thư viện giọng** | Lưu audio mẫu làm giọng người dùng, làm mới danh sách, xóa giọng người dùng. |
| **Nhật ký** | Theo dõi tiến độ và thông báo lỗi. |

Trong tab **Đọc truyện / Clone**:

1. Chọn **VieNeu-TTS v3 Turbo** (mặc định) hoặc **v3 Nano**, chọn thiết bị và nhấn **Tải model**.
2. Chọn **Preset** rồi chọn một giọng có sẵn; hoặc chọn **Clone từ audio** và cung cấp file giọng mẫu. Có thể sử dụng giọng đã train trong `finetune/dataset/raw_audio` hoặc các audio mẫu trong `examples/audio_ref` nếu những thư mục đó có dữ liệu.
3. Dán văn bản hoặc mở TXT/PDF. Bật **Tự tách trước khi tạo** để phân đoạn tự động; dùng **✂ Tách câu thông minh** khi muốn xem trước nội dung đã xử lý.
4. Bấm **🎚 Ngắt nghỉ** để điều chỉnh khoảng dừng. Chọn định dạng **WAV** hoặc **MP3**, đặt Temperature/Max chars/Batch nếu cần.
5. Bấm **▶ Tạo audio**. Thành phẩm nằm trong thư mục xuất đã chọn; có thể bấm **Nghe kết quả**.

**Audio mẫu clone:** ưu tiên bản thu rõ, ít tiếng nền, không có nhạc, âm lượng ổn định và chỉ một người nói. Giọng clone có thể khác nhau tùy chất lượng audio mẫu, model và nội dung văn bản. Chỉ dùng giọng nói khi bạn có quyền hoặc sự đồng ý phù hợp.

## Xử lý nhiều file TXT

Chức năng này thiết kế cho các kịch bản dài và nhiều dự án đọc truyện:

1. Chọn giọng **Preset** hoặc **Clone từ audio** ở tab **Đọc truyện / Clone**, sau đó tải model.
2. Chuyển sang tab **TXT hàng loạt**, bấm **Thêm file TXT** và chọn nhiều file.
3. Có thể xóa hoặc dùng **▲ / ▼** để sắp xếp thứ tự xử lý.
4. Ở ô **Thư mục xuất**, bấm **Chọn…** rồi chọn thư mục trên máy (ví dụ `G:\AUDIO_SLEP`).
5. Giữ bật **Tách câu thông minh cho từng TXT** (mặc định), chọn WAV/MP3 ở tab đọc truyện, rồi bấm **📚 Bắt đầu xử lý**.

Ứng dụng sẽ **tự đọc và tách câu cho từng TXT, xử lý theo đúng thứ tự hàng đợi**, không yêu cầu bấm tách câu thủ công. Mỗi file xuất có tên gốc tương ứng:

```text
Kich_ban_01.txt  →  Kich_ban_01.mp3
Kich_ban_02.txt  →  Kich_ban_02.mp3
```

Trong cùng một lượt, nếu hai file TXT ở các thư mục nguồn khác nhau **có cùng tên**, ứng dụng có cơ chế đưa file trùng sau vào thư mục con mang tên thư mục nguồn. **Lưu ý:** nếu thư mục đích đã chứa audio cùng đường dẫn/tên từ lần chạy trước, file có thể bị ghi đè; hãy sao lưu thành phẩm cần giữ trước khi chạy lại.

Nếu một TXT xảy ra lỗi thông thường, ứng dụng ghi trạng thái lỗi và tiếp tục file tiếp theo. Nút **Dừng sau lượt hiện tại** có hiệu lực ở điểm kiểm tra giữa các lượt xử lý, không thể hủy ngay một lần suy luận của model đang diễn ra.

## Tách câu thông minh và ngắt nghỉ

Ứng dụng có chế độ tách câu thông minh **thủ công** (xem trước bằng nút cắt câu) và **tự động** trước khi tạo. Bộ xử lý cố gắng giữ câu trọn vẹn, dấu kết thúc câu và xuống dòng để ghép âm có khoảng nghỉ phù hợp; với câu quá dài, giới hạn của model vẫn có thể cần người dùng bổ sung dấu câu.

Các khoảng nghỉ mặc định trong **Tkinter**:

| Vị trí | Mặc định |
| --- | ---: |
| Dấu chấm `.` | **0,75 giây** |
| Dấu phẩy `,` | **0,40 giây** |
| Dấu hỏi `?` | **0,80 giây** |
| Dấu chấm than `!` | **0,80 giây** |
| Dấu hai chấm `:` | **0,40 giây** |
| Dấu chấm phẩy `;` | **0,40 giây** |
| Dấu ba chấm `…` | **0,90 giây** |
| Xuống dòng thật | **1,10 giây** |

Mở **🎚 Ngắt nghỉ** để sửa từng giá trị; nút **Đặt mặc định** sẽ phục hồi các mức trên. Nếu câu văn đọc quá gấp hoặc không có điểm dừng như mong muốn, kiểm tra dấu câu trong TXT và tăng khoảng nghỉ tương ứng.

## Chế độ Quality+

**Quality+** là tùy chọn dành cho **Voice Cloning với VieNeu-TTS v3 Turbo**. Chế độ này ưu tiên kiểm tra tính chính xác và độ ổn định của giọng, nhưng **tốn thời gian hơn** chế độ thường:

1. Tiền xử lý audio mẫu: chuyển mono, cắt bớt im lặng, chọn đoạn tham chiếu ngắn, cân âm lượng và có thể khử nhiễu.
2. Giữ cùng thông tin giọng mẫu trong suốt quá trình tạo audio.
3. Sinh **ít nhất hai bản cho mỗi câu**; sinh bản thứ ba khi kết quả tốt nhất vẫn đáng nghi.
4. Dùng nhận dạng lời nói **Whisper large-v3-turbo**, độ tương đồng giọng và tín hiệu méo tiếng để chấm/chọn bản tốt hơn.
5. Cân âm lượng giữa các đoạn, xử lý điểm nối, ghép với thời gian nghỉ theo dấu câu/xuống dòng.
6. Lưu báo cáo cạnh audio, ví dụ **`Kich_ban_01.mp3.quality.json`**, để rà soát đoạn có cảnh báo.

**Yêu cầu quan trọng:** checkpoint `openai/whisper-large-v3-turbo` phải **đã có sẵn trong bộ nhớ đệm local** để Quality+ hoạt động; khi chưa có, chương trình sẽ báo lỗi. Trình khởi chạy tự cài thư viện **không đồng nghĩa với tự tải checkpoint Whisper**. Quality+ chỉ chạy với audio mẫu ở chế độ Clone; không hỗ trợ Preset thuần hoặc Nano. Khi bật Quality+, hệ thống xử lý tuần tự và không sử dụng Batch Size 16 để tăng tốc.

Quality+ giúp phát hiện và giảm một số lỗi thường gặp, **không đảm bảo mọi câu đều đúng tuyệt đối**. Nên nghe kiểm tra lại các đoạn có tên riêng, số liệu, từ địa phương và các đoạn được đánh dấu trong báo cáo. Xem mô tả kỹ thuật tại [docs/quality-plus.md](docs/quality-plus.md).

## Cài đặt mặc định

Các giá trị dưới đây là mặc định trong **giao diện Tkinter** ở `apps/tkinter_app/config.py`; người dùng có thể thay đổi ngay trong ứng dụng:

| Tham số | Giá trị |
| --- | --- |
| Model | VieNeu-TTS v3 Turbo |
| Thiết bị | Auto |
| Độ chính xác | fp32 |
| Temperature | 0,4 |
| Max chars/chunk | 224 |
| Batch Processing | **Bật** |
| Batch size | **16** |
| Denoise audio mẫu | Bật |
| Tự tách câu thông minh | Bật |
| Quality+ | **Tắt** |
| Thư mục xuất mặc định | `outputs/tkinter/` |

Thiết lập được lưu trong **`%USERPROFILE%\.vieneu\tkinter_config.json`** khi đóng ứng dụng. Để khôi phục mặc định, có thể đổi tên file cấu hình này khi ứng dụng đã tắt; lần chạy tiếp theo sẽ dùng lại các giá trị mặc định.

## Xử lý lỗi thường gặp

**1. Nhấp `run_tkinter.bat` nhưng không chạy được:** xem thông báo trong cửa sổ CMD; kiểm tra kết nối Internet, dung lượng trống, phiên bản GPU/driver và thử chạy lại. Nếu lỗi thư viện CUDA, thử chế độ CPU bằng `set VIENEU_INSTALL_MODE=cpu` trong Command Prompt.

**2. Web báo `InvalidPathError: Cannot move ... to the gradio cache`:** đây là lỗi bảo vệ đường dẫn của Gradio khi file ở ngoài thư mục được phép. Kiểm tra phiên bản mã nguồn hiện tại và `allowed_paths` trong cấu hình `launch()`; chỉ thêm thư mục xuất mà bạn thực sự tin cậy. Desktop Tkinter xuất file trực tiếp vào thư mục đã chọn.

**3. Audio nói dính câu / mất khoảng nghỉ:** bật **Tách câu thông minh cho từng TXT**, kiểm tra dấu chấm và xuống dòng trong văn bản, tăng các khoảng dừng ở **🎚 Ngắt nghỉ**.

**4. Đoạn giọng bị lặp, méo, thiếu từ:** thử reference rõ tiếng hơn, giảm Batch Size hoặc tắt Batch để so sánh chất lượng, điều chỉnh Temperature nhẹ và kiểm tra file TXT. Có thể thử Quality+ với Turbo/Clone nếu đã chuẩn bị Whisper local.

**5. Quality+ báo thiếu Whisper:** cần tải sẵn checkpoint `openai/whisper-large-v3-turbo` và các thành phần phụ thuộc tương thích trước khi bật Quality+; xem [tài liệu Quality+](docs/quality-plus.md). Có thể tắt Quality+ để tiếp tục tạo audio thường.

**6. Hết VRAM hoặc tạo chậm:** giảm Batch Size (ví dụ 4–8), thử CPU khi GPU không tương thích, hoặc tắt Quality+ nếu đang ưu tiên tốc độ. Batch Size 16 là **giá trị mặc định**, không phải mức tối ưu cho mọi GPU.

## Cấu trúc dự án và tài liệu

```text
VieNeu-TTS/
├── run_tkinter.bat                # Tự chuẩn bị môi trường và mở Tkinter
├── run.bat                        # Mở Gradio Web (cần .venv đã có)
├── apps/
│   ├── tkinter_app/               # Giao diện Windows và engine desktop
│   ├── gradio_main.py             # Giao diện web
│   ├── batch_files.py             # Xử lý file TXT hàng loạt
│   ├── text_processing.py         # Chuẩn bị và chia văn bản
│   └── quality_plus.py            # Kiểm tra/chọn các bản audio Quality+
├── docs/
│   ├── tkinter-app.md             # Chi tiết giao diện desktop
│   ├── quality-plus.md            # Chi tiết chế độ Quality+
│   └── README_UPSTREAM.md         # README tiếng Anh gốc, lưu tham khảo
├── tests/                         # Các bài kiểm tra phần mềm
├── pyproject.toml                 # Khai báo thư viện
├── uv.lock                        # Phiên bản thư viện được khóa
├── README.vi.md                   # Tài liệu tiếng Việt từ dự án gốc
└── LICENSE                        # Giấy phép dự án
```

Kiểm tra các chức năng liên quan bằng môi trường có `pytest`:

```powershell
uv sync --group dev --extra pdf
uv run python -m pytest tests/test_tkinter_app.py tests/test_batch_txt.py tests/test_sentence_pauses.py tests/test_quality_plus.py
```

Khi chia sẻ mã nguồn lên GitHub, chỉ đưa **mã nguồn, file khởi chạy, cấu hình dependency và tài liệu**; tránh tải lên `.venv/`, `outputs/`, model/checkpoint nặng, file ghi âm cá nhân hoặc dữ liệu riêng tư. `.gitignore` của dự án đã loại trừ một số thư mục sinh ra trong quá trình chạy; hãy kiểm tra lại danh sách file trước khi publish.

## Nguồn gốc và giấy phép

- **Dự án gốc:** [pnnbao97/VieNeu-TTS](https://github.com/pnnbao97/VieNeu-TTS), tác giả được khai báo trong `pyproject.toml`: **Phạm Nguyễn Ngọc Bảo**.
- **Mô hình Turbo:** [VieNeu-TTS-v3-Turbo trên Hugging Face](https://huggingface.co/pnnbao-ump/VieNeu-TTS-v3-Turbo).
- **Giấy phép mã nguồn:** xem [LICENSE](LICENSE) (Apache-2.0). Kiểm tra thêm điều kiện của từng checkpoint/model được sử dụng.
- **Tài liệu upstream:** [README tiếng Anh được lưu lại](docs/README_UPSTREAM.md) và [README tiếng Việt gốc](README.vi.md).

Đây là bản tùy biến phục vụ giao diện desktop, kịch bản dài, điều khiển khoảng nghỉ và kiểm tra chất lượng giọng. Các tính năng Quality+ là lớp xử lý bổ sung trên VieNeu-TTS; **không phải model OmniVoice được tích hợp vào dự án**.
