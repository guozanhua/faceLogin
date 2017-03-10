# faceLogin
刷脸登录
##人脸识别的原理
  其实就是比较两张图片中人脸的相似度，有一个算法来处理。
  官方原话>将两个人脸进行比对，来判断是否为同一个人。支持传两张图片进行比对，或者一张图片与一个已知的face_token比对，也支持两个face_token进行比对。使用图片进行比对时会选取图片中检测到人脸尺寸最大的一个人脸。
## 整体的想法
 １.首先，做出前端的拍照页面。用ＨＴＭＬ５的`video`和`canvas`来模拟拍照，以及将图片绘制在canvas上，发送到后台，调用[face++的API](https://www.faceplusplus.com.cn/)
 来生成×face_token×，这个就相当于一个图片的一个ＩＤ，保存在数据库中。
 ２.  再利用前面的模拟拍照以及绘制图片到后台的解析过程，再将这张图片的face_token和数据库中的对比，得到一个*置信度*，就是一个相识度的检查，*置信度*越高则
 代表同一个人的可能性越大。
##具体步骤
###1. 模拟视频拍摄画面<br>
          <div class="video">
          <video id="video" width="460" height="380" autoplay></video>
          </div>
              //开启摄像头
              var promisifiedOldGUM = function(constraints) {
            // 第一个拿到getUserMedia，如果存在
            var getUserMedia = (navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia);`

            // 有些浏览器只是不实现它-返回一个不被拒绝的承诺与一个错误保持一致的接口
             if (!getUserMedia) {
                return Promise.reject(new Error('getUserMedia is not implemented in this browser-getUserMedia是不是在这个浏览器实现'));
            }`

            // 否则，调用包在一个旧navigator.getusermedia承诺
            return new Promise(function(resolve, reject) {
                getUserMedia.call(navigator, constraints, resolve, reject);
            });

            }

            // 旧的浏览器可能无法实现mediadevices可言，所以我们设置一个空的对象第一
         if (navigator.mediaDevices === undefined) {
                navigator.mediaDevices = {};
            }

            // 一些浏览器部分实现mediadevices。我们不能只指定一个对象
             // 随着它将覆盖现有的性能getUserMedia。.
             // 在这里，我们就要错过添加getUserMedia财产。.     
            if (navigator.mediaDevices.getUserMedia === undefined) {
                navigator.mediaDevices.getUserMedia = promisifiedOldGUM;
            }

         // Prefer camera resolution nearest to 1280x720.
            var constraints = {
                audio: true,
                video: {
                width: 1280,
                height: 720
            }
            };

            navigator.mediaDevices.getUserMedia(constraints)
            .then(function(stream) {
                var video = document.querySelector('video');
                video.src = window.URL.createObjectURL(stream);
                video.onloadedmetadata = function(e) {
                    video.play();
                };
            }).catch(function(err) {
                console.log(err.name + ": " + err.message);
            });
###２．拍照操作（截图）<br>
       `<canvas style="display:none"></canvas>
          将视频截图
         function screenShot(){
            var video = document.getElementById("video");//获取前台要截图的video对象，
            var canvas = document.querySelectorAll('canvas')[0];//获取前台的canvas对象，用于作图
            var ctx = canvas.getContext('2d');//设置canvas绘制2d图，
        	var width = 480;//设置canvas宽
        	var height = 270;//设置canvas高
        	canvas.width = width;
        	canvas.height = height;
        	ctx.drawImage(video, 0, 0, width, height);//将video视频绘制到canvas中
        	images = canvas.toDataURL('image/jpeg');//canvas的api中的toDataURL（）保存图像
        	 $("#addAlert").empty();//清空元素
        	video.pause();
           }
            HTMLCanvasElement.toDataURL() 方法返回一个包含图片展示的 data URI 。可以使用 type 参数其类型，默认为 PNG 格式。图片的分辨率为96dpi。
            如果画布的高度或宽度是0，那么会返回字符串“data:,”。
            果传入的类型非“image/png”，但是返回的值以“data:image/png”开头，那么该传入的类型是不支持的。
             Chrome支持“image/webp”类型
###3.异步发送数据<br>
              //确认登录
            function faceLogin(){
        	if($("#user").val() == undefined || $("#user").val() == "" ||$("#user").val().length == 0 ){
        		$("#addAlert").html("<div id='alert' class='alert alert-warning'><a href='#' class='close' data-dismiss='alert'>&times;</a><p class='text-center'>请输入用户名</p></div>");
        		return;
        	}
        	var userId = document.getElementById("user").value;
        	if(user == undefined || user == "")return;
        	imagesAjax(images,userId);
             }
             //异步提交图片
            function imagesAjax(src,userId) {
			   if(src == "" || src == undefined){
				//toaster.pop({type:"warning",title:"人脸认证提示信息",body:"无人脸",timeout:1500});
				return false;
		   	}
        //window.encodeURIComponent(src)将
        其中的字符转换
        var path = "/appliance/faceLogin?dataUrl=" + window.encodeURIComponent(src)+"&&userId=" + userId;
			$.ajax({
				url: path,
				type: "POST",
				dataType: 'json',
				success: function(re) {
					
					if(re.status == 1){
						document.location.href = "/appliance/login";//跳转到登录页面
						//toaster.pop({type:"success",title:"人脸认证提示",body:re.message,timeout:1500});
					}else{
						$("#addAlert").html("<div id='alert' class='alert alert-warning'><a href='#' class='close' data-dismiss='alert'>&times;</a><p class='text-center'>"+"温馨提示：" + re.message+"</p></div>");
					}
				} 
			});
		    }
###４.　后台接受数据
      /处理图片
			dataUrl = dataUrl.replaceAll("data:image/jpeg;base64,", "");
			BASE64Decoder decoder = new BASE64Decoder();
			try{
				//解密
				byte[] b = decoder.decodeBuffer(dataUrl);
				//处理数据
				for(int i = 0; i < b.length; i++){
					if(b[i] < 0){
						b[i] += 256;
					}
				}
				
				//调用API接口生成face_token
				CommonOperate co = new CommonOperate(API_KEY,API_SECRET,false);
				Response response = co.detectByte(b, 1, "gender,age,smiling,glass,headpose,facequality,blur");
				byte[] re = response.getContent();
				System.out.println(new String(re));
				ObjectMapper objectMapper = new  ObjectMapper();
				HashMap<String,Object> hm = (HashMap<String,Object>)objectMapper.readValue(re, Map.class);  
				ArrayList<HashMap> list = (ArrayList<HashMap>)hm.get("faces");
				
				//数据验证
				boolean flag = true;
				if(list.size() == 0){//没有人脸
					flag = false;
					System.out.println("没有人脸");
					map.put("status", 2);
					map.put("message","没有人脸图像");
					return map;
				}
				String s = "";
				for(Map m : list){
					if(StringUtils.isNotBlank(String.valueOf(m.get("face_token")))){
						s = String.valueOf(m.get("face_token"));
					}
				}
				//查询是否存在用户
				User user =systemService.getUserByLoginId(userId);
			
				if(user == null){
					System.out.println("user为空");
					map.put("status", 2);
					map.put("message","用户Id错误或者不存在");
					return map;
				}
				
				//获取对比置信度
				Response response2 = co.compare(user.getFaceKey(), "", null, s, "", null);
				byte[] re2 = response2.getContent();
				System.out.println(new String(re2));
				HashMap<String,Object> hm2 = (HashMap<String,Object>)objectMapper.readValue(re2, Map.class);  
				double confidence = (double)hm2.get("confidence");
				System.out.println("置信度：" + confidence);
				if(confidence >= 85.00){
					map.put("status", 1);
					map.put("message","成功");
					System.out.println("密码：" + user.getLoginPassword());
					UsernamePasswordToken token = new UsernamePasswordToken();
					token.setUsername(user.getLoginId());
					token.setPassword("1".toCharArray());
					token.setCaptcha(ValidateCodeServlet.gCode);
					SecurityUtils.getSubject().login(token);
					System.out.println("登录完成");
					User login=UserUtils.getUser();
					return map;
				}else{
					map.put("status", 2);
					map.put("message","人脸无法通过验证，请重新验证，或者通过其他方式登录");
					return map;
				}
			}catch(Exception  e){
				System.out.println("报错了" + e.getMessage());
				return null;
			}			
##遇到的问题
    １. 当开启了摄像头时，怎么关闭？只有重新刷新页面来关闭。
    ２. 当发送的url中的dataUrl数据过大时，报错。
      　加上这句‘maxHttpHeaderSize="800000"’，这是请求头过大时出现的问题。
