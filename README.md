基于[Android Volley](https://github.com/mcxiaoke/android-volley)的网络请求工具。  
=====================
####一、说明
Android network manager based on Android Volley, JSON, XML, Map, RequestMap(with file) support.  
AndroidVolley，Android Volley核心库及扩展工程。  
AndroidVolleySample，网络请求工具示例工程。  
Release，AndroidVolley jar包。

####二、Volley基本处理流程：  
1、应用初始化Volley。  
2、Volley创建一个RequestQueue、NetworkDispatcher组及Network。  
3、RequestQueue即一个Request队列，RequestQueue会创建一个ExecutorDelivery。  
4、NetworkDispatcher实质是Thread，从RequestQueue中取Request，通过Network加以执行。  
5、Network负责网络请求处理，具体过程交给HttpStack处理。  
6、HttpStack分HttpURLConnection(SDK_INT>=9)与HttpClient与两种方式。  
7、ExecutorDelivery负责处理请求结果，并与主线程进行交互。  
8、Volley在上述2-7的基础上增加了Cache等附加处理环节。  

####三、网络请求工具  
在AndroidVolley基础上扩展了com.android.http包，增加了ByteArrayRequest及RequestManager，方便JSON、XML、Map<String, Sting>()、及RequestMap<String, File>()的网络请求。  

#####1.初始化RequestManager
	public class VolleyApplication extends Application {
		@Override
		public void onCreate() {
			super.onCreate();
			RequestManager.getInstance().init(this);//初始化工具
		}
	
		@Override
		public void onTerminate() {
			super.onTerminate();
		}
	}

#####2.使用RequestManager
	/**
	 * 测试程序
	 * 
	 * @author panxw
	 *
	 */
	public class MainActivity extends Activity {
	
		private static final String OUT_FILE = "upload.txt";
	
		private static final String OUT_DATA = "sadf464764sdf3ds1f3adsf789213557r12-34912-482130487321gjsaldfalfu2390q3rtheslafkhsdafhreasof";
	
		private static final String POST_URL = "http://allthelucky.ap01.aws.af.cm/memoServer";
	
		private static final String POST_JSON = "{\"action\":\"test\", \"info\":\"hello world\"}";
	
		private static final String GET_URL = "https://raw.githubusercontent.com/panxw/android-volley-manager/master/test.txt";
	
		private static final String UPLOAD_URL = "http://www.splashpadmobile.com/upload.php";
	
		private LoadControler mLoadControler = null;
	
		private ImageView mImageView;
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			this.mImageView = (ImageView) findViewById(R.id.imageView1);
	
			this.testPost();
			this.testGet();
			this.testFileUpload();
			this.testImageLoader();
		}
	
		/**
		 * 测试POST
		 */
		private void testPost() {
			mLoadControler = RequestManager.getInstance().post(POST_URL, POST_JSON,
					requestListener, 0);
		}
	
		/**
		 * 测试GET
		 */
		private void testGet() {
			mLoadControler = RequestManager.getInstance().get(GET_URL,
					requestListener, 1);
		}
	
		/**
		 * 测试POST（文件上传）
		 */
		private void testFileUpload() {
			MainActivity.prepareFile(this);
	
			RequestMap params = new RequestMap();
			File uploadFile = new File(this.getFilesDir(), OUT_FILE);
			params.put("uploadedfile", uploadFile);
			params.put("share", "1");
	
			mLoadControler = RequestManager.getInstance().post(UPLOAD_URL, params,
					requestListener, 2);
		}
	
		/**
		 * 测试图片加载
		 */
		private void testImageLoader() {
			NetworkApplication.getImageLoader().get(
					"http://www.baidu.com/img/bdlogo.png", mImageListener);
		}
	
		/**
		 * 因ImageLoader将回调对象存放在了WeakReference中，所以这里要将ImageListener回调设置成类对象
		 */
		private ImageListener mImageListener = new ImageListener() {
			@Override
			public void onErrorResponse(VolleyError error) {
				System.out.println("Image onErrorResponse");
			}
	
			@Override
			public void onResponse(ImageContainer response, boolean isImmediate) {
				System.out.println("Image onResponse");
				if (response != null && response.getBitmap() != null) {
					System.out.println("Image onResponse daata");
					mImageView.setImageBitmap(response.getBitmap());
				}
			}
		};
	
		/**
		 * 因RequestManager将回调对象存放在了WeakReference中，所以这里要将RequestListener回调设置成类对象
		 */
		private RequestListener requestListener = new RequestListener() {
			@Override
			public void onSuccess(String response, Map<String, String> headers,
					String url, int actionId) {
				System.out.println("actionId:" + actionId + ", OnSucess!\n"
						+ response);
			}
	
			@Override
			public void onError(String errorMsg, String url, int actionId) {
				System.out.println("actionId:" + actionId + ", onError!\n"
						+ errorMsg);
			}
	
			@Override
			public void onRequest() {
				System.out.println("request send...");
			}
		};
	
		@Override
		public void onBackPressed() {
			super.onBackPressed();
			if (mLoadControler != null) {
				mLoadControler.cancel();
			}
		}
	
		private static void prepareFile(Context context) {
			FileOutputStream fos = null;
			try {
				fos = context.openFileOutput(OUT_FILE, Context.MODE_PRIVATE);
				try {
					fos.write(OUT_DATA.getBytes());
				} catch (IOException e) {
					e.printStackTrace();
				}
			} catch (FileNotFoundException e) {
				e.printStackTrace();
			} finally {
				if (fos != null) {
					try {
						fos.close();
					} catch (IOException e) {
						e.printStackTrace();
					}
				}
			}
		}
	
	}
