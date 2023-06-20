思路：分别通过`x264_param_apply_profile`和`x264_param_default_preset` 来测试不同profile与preset下编码耗时与编码出来h264码流大小的实验代码。

profile、preset、tune定义如下：

```c++
const x264_profile_names[] = { "baseline", "main", "high", "high10", "high422", "high444", 0 };

x264_preset_names[] = { "ultrafast", "superfast", "veryfast", "faster", "fast", "medium", "slow", "slower", "veryslow", "placebo", 0 };

x264_tune_names[] = { "film", "animation", "grain", "stillimage", "psnr", "ssim", "fastdecode", "zerolatency", 0 };
```





```c++
  int ret;
  int y_size;
  int iNal_Count = 0;
  int csp = X264_CSP_I420;
  int width = 852, height = 480;
  x264_nal_t* pNals = NULL;
  x264_picture_t* pPic_in = (x264_picture_t*)malloc(sizeof(x264_picture_t));
  x264_picture_t* pPic_out = (x264_picture_t*)malloc(sizeof(x264_picture_t));
  x264_picture_init(pPic_out);
  ret = x264_picture_alloc(pPic_in, csp, width, height);
  x264_param_t* pParam = (x264_param_t*)malloc(sizeof(x264_param_t));
  x264_param_default(pParam);

  std::ofstream output_file;

  for (int i=0; i<6;i++){//6
		for (int j = 0; j < 10; j++) {//10
			for (int k=0; k<8; k++){//8
				//std::ifstream input_file("../cuc_ieschool_640x360_yuv444p.yuv", std::ios::binary);
				std::ifstream input_file("C:/Users/ChengKeKe/Desktop/out.yuv", std::ios::binary);
				std::stringstream out_put_name;
				out_put_name << "out/out_profile_" << x264_profile_names[i] << "_preset_"<< x264_preset_names[j] <<"_tune_"<< x264_tune_names[k]<<".h264";
				output_file.open(out_put_name.str(), std::ios::binary);
				std::cout << "open file" << out_put_name.str() << std::endl;
				//Encode 1000 frame
			//if set 0, encode all frame
				int64_t frame_num = 1000;
				x264_t* pHandle = NULL;
				//Check
				if (!input_file.is_open() || !output_file.is_open()) {
					printf("Error open files.\n");
					return -1;
				}
				ret = x264_param_apply_profile(pParam, x264_profile_names[i]);
				ret = x264_param_default_preset(pParam, x264_preset_names[j], x264_tune_names[k]);
				pParam->i_width = width;
				pParam->i_height = height;
				pParam->i_csp = csp;
				//Param
				//pParam->i_log_level = X264_LOG_DEBUG;
				// pParam->i_threads  = X264_SYNC_LOOKAHEAD_AUTO;
			 // 	pParam->i_frame_total = 0;
				// pParam->i_keyint_max = 10;
			 // 	pParam->i_bframe  = 5;
			 // 	pParam->b_open_gop  = 0;
			 // 	pParam->i_bframe_pyramid = 0;
			 // 	pParam->rc.i_qp_constant=0;
			 // 	pParam->rc.i_qp_max=0;
			 // 	pParam->rc.i_qp_min=0;
			 // 	pParam->i_bframe_adaptive = X264_B_ADAPT_TRELLIS;
			 // 	pParam->i_fps_den  = 1; 
			 // 	pParam->i_fps_num  = 25;
			 // 	pParam->i_timebase_den = pParam->i_fps_num;
			 // 	pParam->i_timebase_num = pParam->i_fps_den;
				pParam->b_repeat_headers = false;//
				pHandle = x264_encoder_open(pParam);
				ret = x264_encoder_headers(pHandle, &pNals, &iNal_Count);
        for (auto nax_index = 0; nax_index < iNal_Count; ++nax_index) {
          output_file.write((char*)pNals[nax_index].p_payload, pNals[nax_index].i_payload);
        }
				y_size = pParam->i_width * pParam->i_height;
				//detect frame number
				if (frame_num == 0) {
					auto start_index = input_file.tellg();
					input_file.seekg(0, std::ios::end);
					auto size = input_file.tellg() - start_index;//获取文件大小
					input_file.seekg(0, std::ios::beg);
					switch (csp) {
						//fseek()和ftell()存在一个潜在的问题就是他们限制文件的大小只能在long类型的表示范围以内，也就是说通过这种方式，只能 打开2,000,000,000字节的文件，不过在绝大多数情况下似乎也已经够用了。
						//如果需要打开更大的文件，你需要用到fgetpos()、 fsetpos()函数了
					
					case X264_CSP_I444:frame_num = size / (y_size * 3); break;
					case X264_CSP_I420:frame_num = size / (y_size * 3 / 2); break;
					default:printf("Colorspace Not Support.\n"); return -1;
					}
				}
				const clock_t begin_time = clock();
				//Loop to Encode
				for (auto frame_index = 0; frame_index < frame_num; frame_index++) {
					switch (csp) {
					case X264_CSP_I444: {
						input_file.read((char*)pPic_in->img.plane[0], y_size);
						input_file.read((char*)pPic_in->img.plane[1], y_size);
						input_file.read((char*)pPic_in->img.plane[2], y_size);
						break; }
					case X264_CSP_I420: {
						input_file.read((char*)pPic_in->img.plane[0], y_size);
						input_file.read((char*)pPic_in->img.plane[1], y_size / 4);
						input_file.read((char*)pPic_in->img.plane[2], y_size / 4);
						break; }
					default: {
						printf("Colorspace Not Support.\n");
						return -1; }
					}
					pPic_in->i_pts = frame_index;

					ret = x264_encoder_encode(pHandle, &pNals, &iNal_Count, pPic_in, pPic_out);
					if (ret < 0) {
						printf("Error.\n");
						return -1;
					}

					//printf("Succeed encode frame: %5d\n", frame_index);

					for (auto nax_index = 0; nax_index < iNal_Count; ++nax_index) {
						output_file.write((char*)pNals[nax_index].p_payload, pNals[nax_index].i_payload);
					}
				}
				//flush encoder
				while (1) {
					ret = x264_encoder_encode(pHandle, &pNals, &iNal_Count, NULL, pPic_out);
					if (ret == 0) {
						break;
					}
					//printf("Flush 1 frame.\n");
					for (auto nax_index = 0; nax_index < iNal_Count; ++nax_index) {
						output_file.write((char*)pNals[nax_index].p_payload, pNals[nax_index].i_payload);
					}
				}

				x264_encoder_close(pHandle);
				std::cout << out_put_name.str() << "cast times : " << float(clock() - begin_time) / CLOCKS_PER_SEC << std::endl;;
				pHandle = NULL;
				input_file.close();
				output_file.close();

			}
		}
  }
  x264_encoder_close(pHandle);
  std::cout << out_put_name.str() << "cast times : " << float(clock() - begin_time) / CLOCKS_PER_SEC << std::endl;;
  pHandle = NULL;
  input_file.close();
  output_file.close();
  x264_picture_clean(pPic_in);
  free(pPic_in);
  free(pPic_out);
  free(pParam);
```

