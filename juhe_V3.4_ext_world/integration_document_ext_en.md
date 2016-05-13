#PEGASI SDK Integration Description Document（world_ext version）
##1、Introduction
Overseas version 3.4, include Native、Banner、Interstitial... Ad style，also include Facebook、Mopub、Yahoo、Admob third-party Native Ad style SDK integration。
##2、Prepare work
###2.1、Add aar
Copy aar file to libs of project，add code to Gradle script。

	repositories {
	    flatDir {
	        dirs 'libs'
	    }
    }
	dependencies {
		compile(name: 'cmadsdk_world_ext_V3.4.1', ext: 'aar')
		notice：integration Admob need to add these aar：
		compile(name: 'play-services-ads-7.8.0', ext: 'aar')
	    compile(name: 'play-services-analytics-7.8.0', ext: 'aar')
	    compile(name: 'play-services-base-7.8.0', ext: 'aar')
	}
	
###2.2、Modify AndroidManifest.xml
Add permission：

	<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
	
other configuration（mopub and admob）：

	<!--for Mopub -->
	 <activity android:name="com.mopub.mobileads.MoPubActivity"
            android:configChanges="keyboardHidden|orientation|screenSize"/>
     <activity android:name="com.mopub.mobileads.MraidActivity"
            android:configChanges="keyboardHidden|orientation|screenSize"/>
     <activity android:name="com.mopub.common.MoPubBrowser"
            android:configChanges="keyboardHidden|orientation|screenSize"/>
     <activity android:name="com.mopub.mobileads.MraidVideoPlayerActivity"
            android:configChanges="keyboardHidden|orientation|screenSize"/>
	<!--for Mopub-->
	
	<!--for admob-->
    <meta-data android:name="com.google.android.gms.version"
            android:value="@integer/google_play_services_version" />	
###2.3、Init SDK(strongly recommended that initialization in the onCreate method of Application)
	Suggesting in OnCreate() method of Application initialize sdk，for example：   
	@Override
	public void onCreate() {
    	super.onCreate();   
   		//The first parameter：Context
    	//The second parameter: Mid(the identifier of the app，the first four numbers of posid)    //The third parameter：product channel id（can be null）
    	CMAdManager.applicationInit(this, "Your AppId", "You channel ID");
	}		

##3. Code Integration
###3.1、 Native	
####3.1.1、load and show Ad
load Ad for example :

	 // The first parameter：Context
	 // The second parameter：Posid
 	 NativeAdManager nativeAdManager = new NativeAdManager(this, "Your posid");
 	 nativeAdManager.setNativeAdListener(new INativeAdLoaderListener() {
		       @Override
		       public void adLoaded() {
				     //get ad
		       		 INativeAd ad = nativeAdManagerEx.getAd();
		       		 if (ad == null) {
                   			 return;
                      }
				    
				     //Initialize the ad view,and use the elemnets of the 
				     //INativeAd object to fill the mAdView
		           	 View mAdView = View.inflate(MainActivity.this,"Your Ad layout", null);
						
		           
		           
		            //Bind the ad with the mAdView
		            //notice: this step is necessary，if don't ,the event like 
		            //click of the ad will not effective.
		             ad.registerViewForInteraction(mAdView);
				
		             if (mAdView != null) {
	                    // remove old mAdView
	                    nativeAdContainer.removeView(mAdView);
                	  }
		            	//add the mAdView into the layout of view container.
		            	//(the container should be prepared by youself)
	            	 nativeAdContainer.addView(mAdView); 
       		  }

            @Override
            public void adFailedToLoad(int i) {
              // load failed
            }


            @Override
            public void adClicked(INativeAd ad) {
            	//ad clicked
            }
        });
     
  		nativeAdManager.loadAd();
  		
 if you want to display Ad :
 
```
 nativeAdManager.showAd();
```
####3.1.2、Native API
- com.cmcm.adsdk.nativead.NativeAdManager
<table>
<tbody>
<tr><td>**Methodology**</td><td>**Method Introduction**</td></tr>
<tr><td>NativeAdManager(Context context, String posid)</td><td>NativaAdManager construct function</td></tr>
<tr><td>setNativeAdListener(INativeAdLoaderListener listener)</td><td>set callback interface</td></tr>
<tr><td>loadAd( )</td><td>concurrent load ads</td></tr>
<tr><td>preloadAd( )</td><td>sequence load ads</td></tr>
<tr><td>getAd( )</td><td>get ads</td></tr>
</tbody>
</table>
- com.cmcm.baseapi.ads.INativeAd
<table>
<tbody>
<tr><td>**Methodology**</td><td>**Method Introduction**</td></tr>
<tr><td>getAdtitle()</td><td>ad title</td></tr>
<tr><td>getAdBody()</td><td>ad description</td></tr>
<tr><td>getAdIconUrl()</td><td>ad icon url</td></tr>
<tr><td>getAdCoverImageUrl()</td><td>ad background picture Url</td></tr>
<tr><td>getAdCallToAction()</td><td>ad button text</td></tr>
<tr><td>getAdStarRating()</td><td>ad star score（can be null）</td></tr>
<tr><td>getAdSocialContext()</td><td>ad download amount or website（can be null）</td></tr>
<tr><td>hasExpired()</td><td> check ad expired（true mean expired）</td></tr>
<tr><td>isDownLoadApp()</td><td>check download type app (true: yes false: no null: no information</td></tr>
<tr><td>setImpressionListener(ImpressionListener listener)</td><td>set callback after SDK ad impression report </td></tr>
<tr><td>registerViewForInteraction(View view)</td><td>bundle ad view with ad itself(requested) </td></tr>
<tr><td>unregisterView()</td><td>unbundling ad view with ad itself(requested)</td></tr>
</tbody>
</table>



###3.2.1、Banner Ad integration
load and show banner Ad for example :
	
	//Parameter：Context
	CMNativeBannerView bannerView = new CMNativeBannerView(this);
	//Set banner Size（requested）
    bannerView.setAdSize(CMBannerAdSize);
	//Set posid（requested）
	bannerView.setPosid(mAdPosid);
	//set callback listener
	bannerView.setAdListener(new CMBannerAdListener() {
        @Override
         public void onAdLoaded(CMAdView ad) {
         	  //after load success and need to display ad must invoke this method 
         	  cmNativeBannerView.prepare();
              mLinearLayout.removeAllViews();
              mLinearLayout.addView(ad);
         }

        @Override
        public void adFailedToLoad(CMAdView ad, int errorCode) {
				//load failed
        }

        @Override
        public void onAdClicked(CMAdView ad) {

        }
    });
    //load ad
    bannerView.loadAd();
    
when Activity destory，suggest destory bannerView

```
 @Override
    protected void onDestroy() {
        super.onDestroy();
        if(cmNativeBannerView != null){
            cmNativeBannerView.onDestroy();
        }
    }
```

####3.2.2、The Main APIs for Native Banner
* com.cmcm.adsdk.banner.CMNativeBannerView

Methodology | Method Introduction
------- | -------
setPosid(String posid) | Set ad position id
setAdSize(CMBannerSize size) |Set banner size
setAdListener(CMBannerAdListener listener) | Set banner callback interface
loadAd() | downloading ad
prepare() | after the success of the load needs to display ads invoke this method

- com.cmcm.adsdk.banner.CMBannerAdSize

Parameter Name | Parameters description
------- | -------
CMBannerAdSize.BANNER_320_50 | 320 * 50 size
CMBannerAdSize.BANNER_300_250 | 320 * 250 size
- com.cmcm.adsdk.banner.CMBannerAdListener



Methodology | Method Introduction
------- | -------
onAdLoaded(CMAdView view) | ad request success
adFailedToLoad(CMAdView view, int error) | ad request failure
onAdClicked(CMAdView view) | ad click

###3.3.1、InterstitialAd integration
primary code for example :

```
//init InterstitialAdManager
//parameter：Context , posid
InterstitialAdManager interstitialAdManager＝new InterstitialAdManager(Context, posid)
//set this callback interface if you need callback。
interstitialAdManager.setInterstitialCallBack(new InterstitialAdCallBack() {
					@Override
					public void onAdLoadFailed(int errorCode) {
						//request failed								
					}
					@Override
					public void onAdLoaded() {
						／／request success
					}

					@Override
					public void onAdClicked() {
						//ad clicked	
					}

					@Override
					public void onAdDisplayed() {
							//ad display
					}

					@Override
					public void onAdDismissed() {
						//click close button that ad destory
					}
				});
```
start load InterstitialAd 

```
interstitialAdManager.loadAd();
```
display InterstitialAd 

```
interstitialAdManager.showAd();
```
###3.3.2、The Main APIs for Native Interstitial

- com.cmcm.adsdk.interstitial.InterstitialAdManager
<table>
<tbody>
<tr><td>**Methodology**</td><td>**Method Description**</td></tr>
<tr><td>loadAd()</td><td>ad downloading</td></tr>
<tr><td>showAd()</td><td>ad display</td></tr>
<tr><td>setInterstitialCallBack(InterstitialAdCallBack listener)</td><td>set callback interface </td></tr>
<tr><td>setInterstialOverClickEnable(boolean enable)</td><td>set whether the shaded are response or not when cliked (no response in default)</td></tr>
</tbody>
</table>
- com.cmcm.adsdk.interstitial.InterstitialAdCallBack
<table>
<tbody>
<tr><td>**Methodology**</td><td>**Method Description**</td></tr>
<tr><td>onAdLoaded( )</td><td>ad request implement</td></tr>
<tr><td>onAdLoadFailed( )</td><td>ad request failure</td></tr>
<tr><td>onAdClicked( )</td><td>ad click</td></tr>
<tr><td>onAdDisplayed( )</td><td>ad display</td></tr>
<tr><td>onAdDismissed(int errorcode)</td><td>click ‘close’ button to close ad unit</td></tr>
</tbody>
</table>
Note: There is no picture loading function of the SDK, need to add the picture loading function from outside. Please set the load function before sending request of interstitial Ads. Please add the following code at the init:
CMAdManagerFactory.setImageDownloadListener(new MyImageLoadListener()); 

##4、Proguard script
Add the following script in your proguard script:
  
###4.1.1、NativeAd Proguard script
	-dontwarn com.cmcm.adsdk.**
	-dontwarn com.cleanmaster.**
	-dontwarn com.facebook.ads.**
	-keep class com.facebook.ads.**{*;}
	-keep class  com.cmcm.adsdk.** { *;}
	-keep class  com.cleanmaster.** { *;}
    
###4.1.2、add Mopub Proguard script

	-keep class  com.mopub.nativeads.*{*;}
	-keep class  com.mopub.common.**{*;}
	-keep class  com.mopub.network.**{*;}
	-dontwarn com.mopub.**
	-keep class com.cmcm.adsdk.nativead.MopubNativeAdLoader{
    	    <fields>;
     	    <methods>;
	}
	-keep class com.mopub.mobileads.MoPubActivity{
      	    <fields>;
			<methods>;
	}
	-keep class com.mopub.mobileads.MraidActivity{
    	    <fields>;
			<methods>;
	}
	-keep class com.mopub.common.MoPubBrowser{
    	    <fields>;
			<methods>;
	}
	-keep class com.mopub.mobileads.MraidVideoPlayerActivity{
    	    <fields>;
			<methods>;
	}

###4.1.3、Admob Proguard script

	-keep class com.cmcm.adsdk.nativead.AdmobNativeLoader{
  	      <fields>;
  	      <methods>;
	}
	-keep public class com.google.android.gms.ads.**{
   		public *;
	}
	# For old ads classes
	-keep public class com.google.ads.**{
   		public *;
	}

	# For mediation
	-keepattributes *Annotation*
	# Other required classes for Google Play Services
	# Read more at http://developer.android.com/google/play-services/setup.html
	-keep class * extends java.util.ListResourceBundle {
   		protected Object[][] getContents();
	}
	-keep public class com.google.android.gms.common.internal.safeparcel.SafeParcelable {
   		public static final *** NULL;
	}
	-keepnames @com.google.android.gms.common.annotation.KeepName class *
	-keepclassmembernames class * {
   		@com.google.android.gms.common.annotation.KeepName *;
	}
	-keepnames class * implements android.os.Parcelable {
   		public static final ** CREATOR;
	}
###4.1.4、Yahoo Proguard script
   
     -keep class com.cmcm.adsdk.nativead.FlurryNativeLoader{
        <fields>;
        <methods>;
     }
	keep class com.flurry.** { *; }
    dontwarn com.flurry.**
    keepattributes *Annotation*,EnclosingMethod
    keepclasseswithmembers class * {
       public <init>(android.content.Context, android.util.AttributeSet, int);
    }
    #If you are using the Google Mobile Ads SDK, add the following:
    # Preserve GMS ads classes 
    keep class com.google.android.gms.ads.** { *; }
    dontwarn com.google.android.gms.ads.**
    #If you are using the Google Play Services
    # Preserve GMS classes to enable fetching the Advertising ID 
    keep class com.google.android.gms.**

	
##5、Error Code

ErrorCode | Instruction
------- | -------
10001 | lack the settings of the ad position，(perhaps lacks sdk initialization，or the cloud end closes the ad position）
10002 | lacks ad data
10003 | internal error
10004 | request timeout
10005 | There is no this type of advertising
