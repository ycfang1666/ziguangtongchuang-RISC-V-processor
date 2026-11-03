RV32五级流水线处理器工程（v008，180MHz，含在线逻辑分析探针）

使用环境
Pango Design Suite 2022.2-SP6.4；Logos2 / PG2L200H / FBB676 / -6。
工程为独立RTL工程，已包含运行固件的ROM初始化镜像。仅生成FPGA位流不需要Python、RISC-V编译器或其他工程目录。

目录
rv32_cpu_v008.pds       PDS工程入口，位于根目录
rtl/                   全部Verilog设计源文件，顶层rv32_cpu_top
constraint/            引脚/时钟约束rv32_cpu.fdc和探针配置rv32_cpu.fic
firmware/firmware.hex   64KiB代码ROM初始化镜像，PDS自动读取
firmware/firmware.elf   与镜像对应的RISC-V可执行文件
firmware/src/          启动程序、串口监控、CoreMark移植层和链接脚本
firmware/coremark/      官方CoreMark源码和许可证
scripts/               可选的固件重编译工具
docs/                  源文件校验记录、探针位域及工程构建验证记录

PDS图形界面编译
1. 将整个工程目录解压到可写目录，保留各子目录，不单独移动.pds文件。
2. 在PDS中选择File -> Open Project，打开根目录rv32_cpu_v008.pds。
3. 顶层、器件、ADS综合器、引脚/时钟约束和FIC均已配置。
4. 在流程树运行Generate Bitstream，依次完成Compile、Synthesize、Device Map、Place & Route和位流生成；同时检查Report Timing是否满足约束。
5. 输出位流为generate_bitstream/rv32_cpu_top.sbit。PDS生成的compile、synthesize、device_map、place_route、report_timing等目录均为构建产物。
6. 位流配置电压选项SCBV已保存为High Voltage，无需重新添加源文件或手工编辑路径。

构建验证
本工程已复制到另一个带空格的目录，通过原生.pds入口完成全流程构建。
180MHz多角时序及全部板级引脚通过，资源为7604 LUT、7775 FF、67 DRM、4 DSP。
重编译位流与已测v008仅生成日期/时间头字段的8个字节不同，其余配置数据逐字节一致。
固件由迁移后的C/汇编源码重新编译，全部65536字节ROM与随附镜像一致。
本目录交付源文件和ROM镜像，不附带中间数据库；构建验证细节见docs/build-validation.json。

板卡接口与运行
输入clk=D18、27MHz；低有效复位rstn=C22；LED=A20，高有效。
远程平台串口FPGA RX=AC26、TX=AB26，115200/8N1，LVCMOS33/3.3V。
PLL将CPU和片内存储时钟设为180MHz。上电后串口进入命令提示符。
help查看命令，coremark运行自动迭代、至少10秒的正式测试；result重发最近完整报告。
pace N设置所有串口输出的分段间隔（0..1000ms，默认250ms）。
其他命令包括led 0/1、time、selftest、stats。固件版本标记为v008。

在线逻辑分析
探针配置constraint/rv32_cpu.fic：240位、2048点，CPU时钟域注册采样。
平台需要FIC时，导入本工程对应文件；本地调试使用本次编译得到的位流。
远程平台手动采样会阻塞串口页面，应先在串口启动coremark，再切换逻辑分析界面开始采样。
探针位域见docs/probe-map.json。

处理器范围
五级单发射RV32I+Zmmul，支持M态CSR、异常和中断、64项BTB、两路I/D Cache。
支持MUL/MULH/MULHSU/MULHU，DIV/REM作为非法指令处理；不声明完整RV32IM。
代码ROM和数据RAM各64KiB。官方CoreMark源码保持不变。

可选：修改并重新编译固件
安装RISC-V GCC15.2.0后，可执行：
python scripts/build_firmware.py --gcc "交叉编译器路径/riscv-none-elf-gcc.exe"
默认生成firmware/rebuilt下的ELF/HEX/MAP和校验信息，不替换随工程提供的镜像。
需要使用修改后的固件时，将新的firmware.hex替换firmware/firmware.hex，再从Compile开始重新构建PDS工程。
ISA/ABI/主要选项：-march=rv32i_zmmul -mabi=ilp32 -O2 -fno-builtin -fno-unroll-loops。
