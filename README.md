RISC-V处理器与YOLO系统提交工程

正式普通版：01_upload/plain/cpu_yolo_162m.sbit，主频162MHz。
当前调试版：01_upload/ila，178位简易ILA，必须使用同目录SBIT、FDC、FIC、ADF。
普通版来源peermapped162_perf，调试版来源easyila162_snapshot。两者均沿用冻结位流，未重新布局布线。

02_source包含选定版本硬件、固件、模型、第三方来源、预编译镜像与重建脚本。03_reports为对应实现记录；04_evidence保留原始上板与本地验证记录；05_docs为技术说明。历史文件中的路径和board_tested标志按记录产生时状态保留，不作为当前操作说明。

远程演示采用115200/8N1串口：等待YOLO_READY后，Q查询、C跑5000轮CoreMark、D运行内置图像。当前远程平台不演示外部图像上传，H协议和电脑叠框仅有本地验证。设备融合及独立准确率仍需补验收。

重建和自检见02_source/README.txt。请先执行verify.py核对全包哈希。
