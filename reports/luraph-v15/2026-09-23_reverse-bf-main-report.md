# Phân tích tiếp bf_main.lua

Ngày: 2026-09-23. Phạm vi: [scope.md](artifacts/bf_main/scope.md).

## Kết quả

Đã giải mã và giải nén cả hai blob, kiểm chứng từng byte bằng một decoder độc lập. Lần tiếp tục đã đọc được prototype root bằng harness decoder dữ liệu, gỡ lớp XOR trên 4.977 dòng dữ liệu lệnh và đối chiếu hai phép tự sửa opcode. Chưa phục dựng toàn bộ Lua gốc hoặc chạy gameplay. `dec_g` chứa bootstrap, VM và nhiều hàm gameplay đọc được; `dec_E` vẫn chưa được giải đầy đủ.

| Gate | Verdict | Bằng chứng |
|---|---|---|
| Source identity | VERIFIED | SHA-256 và kích thước trong manifest |
| Base85 expansion/compaction | VERIFIED_FOR_THIS_SAMPLE | Hai alphabet đều 35..119; viết tại chỗ và buffer riêng khớp |
| LZMA output | INDEPENDENT_BYTE_IDENTICAL | Port O và Python liblzma khớp cả hai output, EOF hợp lệ |
| E initial envelope | TWO_IMPLEMENTATIONS_AGREE | Port state machine và decoder số nguyên trực tiếp khớp |
| Root block XOR | INDEPENDENT_BYTE_IDENTICAL | Port JS khớp block 27.068 byte do q trong harness Luau tạo |
| Root prototype | Q_RETURNED_FACTORY_BLOCKED | Factory 2801; 5.018 dòng dữ liệu lệnh; chặn trước factory |
| Root instruction XOR | TWO_IMPLEMENTATIONS_AGREE | 4.977 dòng được biến đổi; đối chiếu cả 5.018 dòng với arithmetic Luau riêng |
| Root bootstrap | BOUNDED_STATIC_BOOTSTRAP | Hai self-modifier khớp Luau; dừng ở PC 3/opcode 118 |
| Uniform record layout | HYPOTHESIS_REJECTED | 60/450 candidate vượt giới hạn buffer |
| Full prototype/opcode recovery | UNRESOLVED | Chưa hoàn thành |
| Gameplay/runtime behavior | NOT_RUN | Chỉ chạy decoder dữ liệu và arithmetic riêng; không chạy VM gốc/gameplay/Roblox |

## Artifact và hash

Các khoảng offset trong báo cáo dùng zero-based `[start,end)`.

| Artifact | Byte | SHA-256 |
|---|---:|---|
| `bf_main.lua` | 1,173,560 | `897ee76219faf76adb276d7c62168b62eaa90a2d77d3d0de9a8884c0ab981f0c` |
| [dec_g.lua](artifacts/bf_main/dec_g.lua) | 435,924 | `9eb2194c9a51648b991a6fb34e15f01353114f4cb571cca67e7e1cba77b7c4ec` |
| [dec_E.bin](artifacts/bf_main/dec_E.bin) | 845,745 | `4e8818f7d6cb28b9145bed26c576c686f889af4d7b418d3d08f06c308349309b` |
| [E.payload.bin](artifacts/bf_main/E.payload.bin) | 92,516 | `59474f63beb68810af696a51aac7f73ebfd4f3d5c6ad81b7726c2a51737c9a5b` |
| [root.block.bin](artifacts/bf_main/root.block.bin) | 27,068 | `d65760495a18cbcf6ea82ca2c861952555c351b2e356cab95054cda371cb0c3c` |

`dec_g.readable.lua` chỉ thêm xuống dòng ngoài string sau dấu `;`; dùng `dec_g.lua` làm artifact chuẩn. Inventory trong `g.index.json` là phép tách lexical theo mẫu của file này, không phải kết quả parse AST Luau đầy đủ.

## F-01: Pipeline ngoài đã được kiểm chứng

severity: n/a_re; evidence_ids: E-01, E-02, E-03; confidence: high; status: VERIFIED; location: original H/O.

`g` lấy từ `[1072798,1170883)`, E từ `[104,1069961)`. Sau bỏ bốn byte wrapper và thay thế mười ký tự:

| Blob | Trước expansion | Sau expansion | Sau 5→4 | Port O đọc | Output thực ghi |
|---|---:|---:|---:|---:|---:|
| g | 98,085 | 98,105 | 78,484 | 78,477 | 435,924 |
| E | 1,069,857 | 1,069,885 | 855,908 | 855,900 | 845,745 |

Mọi digit sau expansion đều trong 35..119. Giá trị sau trừ 35 là digit 0..84. Thứ tự trọng số từ cao xuống thấp là `b4,b1,b2,b3,b0`; output ghi u32 little-endian. Không có quintet tràn u32 trong mẫu này. `R` được tạo lại trước E; số cache hit lần lượt là 0 và 4. Đây là memoization phép chuyển quintet, không có cache dùng chung g→E.

Port O chặn input thiếu, output tràn và nhánh kết thúc sớm. Python `lzma` giải mã độc lập với `FORMAT_RAW`, `FILTER_LZMA1`, `lc=3,lp=0,pb=0`, dictionary 16 MiB: cả hai output khớp từng byte và `eof=true`. Dictionary này là cấu hình kiểm chứng đủ lớn, không phải thông số dictionary được chứng minh từ header.

Port O dừng khi ghi đủ f byte; liblzma còn đọc EOS. Vì vậy 7/8 byte chưa đọc của port không đồng nghĩa toàn bộ là padding. Sau EOS, liblzma còn 1 byte ở g và 2 byte ở E.

Không cần gọi đây là LZMA tùy biến để giải thích mẫu: LZMA1 chuẩn giải nén được. Không có cơ sở gọi phép mã hóa ngoài là mã hóa mật mã. Kết luận này không loại trừ các phép biến đổi bên trong E.

## F-02: g chứa cả logic gameplay đọc được

severity: n/a_re; evidence_ids: E-02, E-04; confidence: high for cited source; status: STATIC_OBSERVED; location: dec_g numeric fields.

Inventory lexical nhận diện 137 numeric function entries và 234 named function entries. Không đồng nhất 137 mục số với 137 opcode hay 137 hàm native: trong đó có cả bộ thực thi VM lớn.

Các đoạn đã tách riêng:

- [field 12](artifacts/bf_main/fields/12.lua), offset `[853,1153)`: tạo danh sách từ `Workspace.Enemies`, tùy chọn thêm `Characters`.
- [field 87](artifacts/bf_main/fields/87.lua), `[1154,1353)`: tính khoảng cách, trả vô cực nếu nhân vật không sống hoặc thiếu HRP.
- [field 85](artifacts/bf_main/fields/85.lua), `[433007,434574)`: code truy vấn danh sách server Roblox, đọc/ghi `Banana Cat Hub/NotSameServers.json`, chọn server ít người và gọi remote teleport.
- [field 950](artifacts/bf_main/fields/950.lua), `[434596,435378)`: lọc, chuẩn hóa, loại trùng và sắp xếp tên boss qua `BossRuntime`.

Đây là hành vi hiện diện trong source. Chưa chứng minh các closure này được gọi trong một phiên thực tế, và chưa lập đủ danh mục endpoint của toàn bộ script.

## F-03: Envelope đầu E đã có decoder tái hiện

severity: n/a_re; evidence_ids: E-05, E-06; confidence: high for this input; status: VERIFIED_BY_TWO_PORTS; location: oP and its helper methods.

`dec_g` kết thúc bằng `:oP(...)`. `oP` bắt đầu tại state 27, đọc buffer E rồi gọi `q`; sau `q` nó chọn factory qua `b[M[M[2]]](b,M,nil,b.a,nil)` và trả closure. File ngoài tiếp tục gọi giá trị H trả về bằng `:H()(...);`.

| Miền E | Nội dung |
|---|---|
| `[0,2)` | Entry index = 187 |
| `[2,3)` | Flag = false |
| `[3,6)` | Giá trị 45,848; trừ 45,398 thành 450 mục |
| `[6,1389)` | Bảng tra 450 số nguyên; chưa áp một ý nghĩa duy nhất cho mọi mục |
| `[1389,1392)` | Kích thước buffer tiếp theo = 92,516 |
| `[1392,93908)` | Buffer được copy vào environment slot 4 |
| `[93908,845745)` | 751,837 byte còn lại; chưa phân tích đầy đủ |

State machine đặt bảng tra vào slot 5, flag vào slot 7 và cập nhật host slot 118 thành 93,908. Slot 118 trước đó bằng 0.

Codec số nguyên tại envelope không phải ULEB128 thông thường. Gọi a,b,c,d là các byte liên tiếp:

```text
a < 128: a
b < 128: (a-128)*128 + b
c < 128: (a-128) + (b-128)*16384 + c*128
otherwise: (a-128)*16384 + (b-128)*128
         + (c-128)*2097152 + d%128 + (d-d%128)*2097152
```

Chỉ đọc byte kế tiếp nếu các byte trước có bit cao. Dạng bốn byte sử dụng toàn bộ byte cuối. Không tự suy rộng codec này cho mọi trường bên trong q: bước thử ở F-04 đã bác bỏ việc áp một layout duy nhất lên mọi mục.

## F-04: Mốc phân tích tĩnh trước harness

Phần này lưu kết quả của mốc trước. Giới hạn về việc chưa chạy q và chưa đối chiếu block root đã được giải quyết trong F-05; giả thuyết layout đồng nhất vẫn bị bác bỏ.

severity: n/a_re; evidence_ids: E-04, E-07; confidence: medium; status: PARTIAL_STATIC; location: q helpers Le/K/g/Tt/ne/i/Re/ke.

Bảng tra mục 187 là 3363. Nhánh đầu của q đọc từ offset 3364 trong buffer con. Lần theo source cho đường trạng thái `362 → 42 → 13 → 470 → 233 → 117 ↔ 376 → 178`:

- Block mã hóa bắt đầu tại 3367, dài 27,068 byte, kết thúc tại 30,435.
- State ban đầu `(175 + 187) % 256 = 106`.
- Với mỗi byte: `state = (state*237 + 15) % 256`; `plain = cipher XOR state XOR 175`.
- Đã tạo `root.block.bin`; chưa chạy toàn bộ q để xác nhận toàn bộ prototype/constant/opcode hoặc đối chiếu độc lập block này.

Các VM handler có phép XOR `339358108` trên mục lookup trước khi đọc record. Thử chuẩn hóa 91 giá trị lớn bằng hằng số này và áp cùng layout root cho 450 mục cho ra 60 candidate vượt buffer, đồng thời có overlap/gap. Kết quả **UNIFORM_RECORD_LAYOUT_HYPOTHESIS_REJECTED**; 390 candidate nằm trong giới hạn cũng chưa được coi là record đúng.

Ở mốc này, việc còn lại là nhánh q sau state 178, các kiểu entry/codec và các phép biến đổi trước khi lập schema. Lần tiếp tục bên dưới tiến xa hơn bằng decoder dữ liệu cô lập; vẫn chưa có bản Lua gốc được phục dựng hoàn chỉnh.

## F-05: Prototype root và block dữ liệu được kiểm chứng

severity: n/a_re; evidence_ids: E-08, E-11; confidence: high for this sample; status: Q_RETURNED_FACTORY_BLOCKED; location: oP → q → factory 2801.

Harness xây table host từ AST của `dec_g.lua`, thay cả 137 numeric function factories bằng stub dừng trước khi compile, bỏ lời gọi top-level gốc và giới hạn global. Decoder oP/q chạy offline đến stub `STOP_BEFORE_FACTORY`; không chạy factory 2801 hoặc closure gameplay. Các số đọc được:

| Trường | Giá trị |
|---|---:|
| Entry lookup | 187 |
| Factory | 2801 |
| Entry PC | 4994 |
| Initial dispatch mode | 2 |
| Register capacity | 163 |
| Dòng trong mỗi mảng O/B/p/X | 5018 |
| Host cursor sau q | 93908 |

`q_buffers/buffer2.bin` khớp từng byte `root.block.bin`, SHA-256 `d65760495a18cbcf6ea82ca2c861952555c351b2e356cab95054cda371cb0c3c`. Bảng lookup 450 mục cũng khớp envelope JS. Đây là kiểm chứng decoder/prototype trên mẫu này; không chứng minh mọi nhánh q hoặc mọi entry đều đã được khôi phục. Cursor vẫn ở 93908 nên phần đuôi E chưa được giải bởi lượt root này.

## F-06: Lớp XOR của lệnh và đường khởi động

severity: n/a_re; evidence_ids: E-09, E-10, E-11; confidence: high for tested arithmetic, bounded for control flow; status: BOUNDED_STATIC_BOOTSTRAP; location: factory 2801, initial mode 2.

AST cho thấy factory có bốn mode `2,171,51,81`. `root.handlers.json` giữ các đoạn source sau khi rút gọn điều kiện dispatch cho giá trị opcode 0..255 của từng mode. Cả 1.024 excerpt compile được; **không suy ra mỗi mode có 256 opcode hợp lệ** vì giá trị ngoài miền thật có thể rơi chung nhánh cuối. Chưa kiểm chứng độc lập ngữ nghĩa toàn bộ excerpt.

Ở PC 4999, mode 2/opcode 34 biến đổi O/B/p/X tại PC 1..4977 bằng `value XOR ((2 XOR i) AND 127)` với `i=1..4977`, rồi đặt opcode tại PC 4999 thành 92. `root.unmasked.json` chỉ là dữ liệu sau lớp này, chưa phải disassembly hoàn chỉnh. Oracle Luau chỉ chạy excerpt arithmetic này trên mảng số; toàn bộ 5.018 dòng khớp kết quả JS.

Đường điều khiển đã lần tĩnh: `4994 → 4995 → 4997 → 4999 → 5000 → 5001 → 4978 → 17 → 19 → 21 → 23 → 4978 → 4968 → 4970 → 4972 → 4974 → 4975 → 4977 → 4978 → 1 → 3`. PC 23 và PC 3 được xét lại sau khi tự sửa, như log đầy đủ trong `root.bootstrap.json`.

| PC | Trước: opcode, O, B, p | Sau: opcode, O, B, p |
|---|---|---|
| 23 | 107, 42, 9884, 4936 | 124, 0, 0, 4977 |
| 3 | 91, 88, 31, 7852 | 118, 62, 63, 62 |

Cả hai phép sửa khớp arithmetic Luau chạy riêng. Bộ đánh giá JS chỉ chấp nhận AST arithmetic, helper W1/A1 và đọc/ghi mảng lệnh tại PC hiện tại; phần lần đường chỉ nhận các mẫu move/load/jump cụ thể. Nó dừng trước opcode 118 ở PC 3 vì thao tác đóng gói thanh ghi chưa được hỗ trợ. Đây là giới hạn bộ phân tích hiện tại, không phải lỗi của mẫu hoặc bằng chứng không thể khôi phục tiếp.

Việc tiếp theo: hỗ trợ thao tác dữ liệu của opcode 118, tiếp các lần đổi mode/tự sửa có thể gặp, rồi nối operand với constant/prototype lazy qua bằng chứng sử dụng thật. `constants.sweep.json` từ lượt thăm dò trước chưa được xác minh lại trong lần này; không dùng các entry lỗi hoặc offset còn mã hóa để kết luận format hỏng.

## Evidence và đường tái hiện

| ID | source_ref | repro_command | content_hash |
|---|---|---|---|
| E-01 | bf_main.lua | `Get-FileHash bf_main.lua -Algorithm SHA256` | Hash nguồn ở bảng artifact |
| E-02 | manifest.json, dec_g.lua, dec_E.bin | `node scripts/unpack_bf.mjs` | Hash output trong manifest và bảng artifact |
| E-03 | independent_verification.json, checks.json | `python scripts/verify_lzma.py`; `node scripts/verify_unpack.mjs` | n/a, kết quả chạy kiểm chứng các hash E-02 |
| E-04 | g.index.json, fields/*.lua | `node scripts/index_g.mjs`; `node scripts/show_fields.mjs 12 87 85 950 Le K g Tt ne i Re ke` | Các đoạn source tham chiếu dec_g có hash cố định |
| E-05 | envelope.json, E.payload.bin | `node scripts/trace_envelope.mjs` | Hash E.payload ở bảng artifact |
| E-06 | envelope_verification.json | `node scripts/verify_envelope.mjs` | n/a, so sánh dữ liệu E-05 |
| E-07 | records.json, root.block.bin | `node scripts/decode_records.mjs` | Hash root.block ở bảng artifact |
| E-08 | q.graph.json, q.summary.json, q.harness.meta.json | `node scripts/run_q_harness.mjs root`; `node scripts/summarize_q.mjs` | Hash artifact trong root.verification.json |
| E-09 | root.handlers.json, root.unmasked.json | `node scripts/disassemble_root.mjs` | unmasked: `c38cbb341474498c3c235110f581ec463eaedeeea109a85def81c23e662b984f` |
| E-10 | root.bootstrap.json | `node scripts/trace_root_bootstrap.mjs` | `900a0bd1205b98a350d5c96b1c3c5ac90a2ca6bb575105c989f7635a6293d02c` |
| E-11 | root.verification.json, root.arithmetic-oracle.lua/stdout | `node scripts/verify_root.mjs` | Manifest hash từng artifact; 9 checks PASS |

P-01, path_type=callflow của source gốc: input E/g → strip/expansion → base85 → LZMA1 → `loadstring(dec_g)(dec_E)` → oP envelope → q → chọn factory → closure → lời gọi cuối `(...);`. E-01/E-02/E-03 chứng minh đến đầu ra giải nén; E-04/E-05/E-06 chứng minh source và data envelope; E-07 ghi giả thuyết layout thất bại. Harness E-08 dừng ở stub factory, không đi tiếp đường gameplay.

P-02, path_type=solve: root prototype từ E-08/F-05 → chuyên biệt AST dispatch E-09 → gỡ XOR ở PC 4999 → lần move/load/jump và arithmetic tự sửa E-10/F-06 → dừng trước opcode 118. E-11 đối chiếu block, mảng XOR và hai self-modifier, cùng kiểm tra cú pháp các excerpt.

## Chạy lại và kiểm thử

Cần Node.js và Python có thư viện chuẩn `lzma`; không cần Roblox hoặc thư viện bên thứ ba. Trong phiên này dùng Node 24.19.0 và Python 3.13 tại đường dẫn cài đặt local.

Phần tiếp tục dùng thêm các binary Luau có sẵn trong `tools/luau-0.739/`. Lượt này không tải hoặc cập nhật công cụ.

```powershell
node scripts/unpack_bf.mjs
& 'C:\Users\LENOVO\AppData\Local\Programs\Python\Python313\python.exe' scripts/verify_lzma.py
node scripts/verify_unpack.mjs
node scripts/index_g.mjs
node scripts/show_fields.mjs oP St Yt mt Lt Ut dt vt Rt u1 k1 I1 t1 q c Le K g Tt ne i Re ke 12 87 85 950
node scripts/trace_envelope.mjs
node scripts/verify_envelope.mjs
node scripts/decode_records.mjs
node scripts/parse_g_ast.mjs
node scripts/run_q_harness.mjs root
node scripts/summarize_q.mjs
node scripts/trace_root_bootstrap.mjs
node scripts/verify_root.mjs
```

Kiểm thử đã PASS: compaction tại chỗ khớp buffer riêng; decoder không cần tail còn nguyên sau compaction; input bị cắt ngắn bị từ chối; yêu cầu thêm một byte output sau EOS bị từ chối. Source hash sau phân tích không đổi.

Timeline ngày 2026-09-23: cố định mẫu → port H/O offline → đối chiếu liblzma → inventory source g → port và đối chiếu envelope → biến đổi block root → bác bỏ layout record đồng nhất → ghi báo cáo ban đầu. Lần tiếp tục: kiểm tra harness đã có → chạy lại decoder root với factory bị chặn → rút gọn AST dispatch → gỡ lớp XOR đầu → lần bootstrap và hai self-modifier → kiểm chứng độc lập arithmetic, 9 checks PASS → cập nhật báo cáo. Không phát sinh request đến target; chỉ thực thi decoder dữ liệu và arithmetic cô lập, không thực thi VM gốc hoặc gameplay.
