#  Concepets Implemented: 📱
- Calling API using Volley
- Glide library 
- Implicit intent data to share in different apps

# Screenshots - 
<img src="https://github.com/JyotiJauhari/Share-Memes/blob/master/Screenshots/main%20screen.jpeg" width = "200" height = "400"  >	 

# How I made it 🤔
## Simple UI - Not a big deal
- Image View
- two buttons share and next
- progress bar
[XML file Link](https://github.com/JyotiJauhari/Share-Memes/blob/master/ShareMemesApp/app/src/main/res/layout/activity_main.xml)

## Manifest file code :
- Just a single permission for using internet
`<uses-permission android:name="android.permission.INTERNET"/>`
 
## Gradle dependencies:

    implementation 'com.android.volley:volley:1.1.1'
    implementation 'com.github.bumptech.glide:glide:4.11.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
For API Calling I have used Volley library , so need to add dependencies for it . Also as image is fetched from a url  so glide librarary is used because it is " An image loading and caching library for Android focused on smooth scrolling " .

## Kotlin code:
 - Two .kt files
 - MainActivity.kt
 - MySingleton.kt

**1. MainActivity.kt :**
create three functions :
 - loadMeme()
 - shareMeme()
 - nextMeme()

**nextMeme( ) :**
*A function used to open nextMeme. Just need to call loadMeme( ) function in it.*

**shareMeme( ) :**
*A function used to share Meme .
Done using implicit intent.*

    fun  shareMeme(view:  View) {
	    val intent =  Intent(Intent.ACTION_SEND)
        val intent = Intent(Intent.ACTION_SEND)
        // intent.type - to share text (if wanna share image use jpg, music-mp3, if dont mention the type then it wont show u the list of option apps to share)
        intent.type = "text/plain"
        intent.putExtra(Intent.EXTRA_TEXT,"Hey, checkout this awesome meme I got from reddit $currentImageUrl")
        val chooser = Intent.createChooser(intent, "Share this app using..")
        startActivity(chooser)
        }


**loadMeme( ) :**
*A function used to send request using volley and load meme into current screen.*

		// private func so cant be accessed outside this class
        private fun loadMeme(){
        //progress bar to be shown when image is loading
        progressBar.visibility = View.VISIBLE
        // Instantiate the RequestQueue.
        //val queue = Volley.newRequestQueue(this)

		//api link - used an open source api (https://github.com/R3l3ntl3ss/Meme_Api)
        val url = "https://meme-api.herokuapp.com/gimme"

        // Request a jsonObject response from the provided URL.
        val jsonObjectRequest = JsonObjectRequest(
            Request.Method.GET, url, null,
            Response.Listener { response ->
             currentImageUrl = response.getString("url")

                //using glide lib (https://github.com/bumptech/glide)
                Glide
                    .with(this)
                    .load(currentImageUrl)
                    .listener(object : RequestListener<Drawable>{

                        //onLoadFailed
                        override fun onLoadFailed(
                            e: GlideException?,
                            model: Any?,
                            target: Target<Drawable>?,
                            isFirstResource: Boolean
                        ): Boolean {
	                        //progress bar to be hided if loadFailed
                            progressBar.visibility= View.GONE
                            return false
                        }

                        //onResourceReady
                        override fun onResourceReady(
                            resource: Drawable?,
                            model: Any?,
                            target: Target<Drawable>?,
                            dataSource: DataSource?,
                            isFirstResource: Boolean
                        ): Boolean {
	                        //progress bar to be hided if image loaded
                            progressBar.visibility = View.GONE
                            return false
                        }
                    }


                    )
                    .into(meme_ImageView);
            },
            Response.ErrorListener {

            })

        // Add the request to the RequestQueue.
        //queue.add(jsonObjectRequest)
		

		//add to request queue which is in MySingleton Class
        MySingleton.getInstance(this).addToRequestQueue(jsonObjectRequest)
    }

**2. MySingleton.kt :**

*singleton class - encapsulates `RequestQueue` and other Volley functionality.*

 *A key concept is that the `RequestQueue` must be instantiated with the Application not an Activity context. This ensures that the `RequestQueue` will last for the lifetime of your app, instead of being recreated every time the activity is recreated (for example, when the user rotates the device).*

    
    class MySingleton constructor(context: Context) {
    companion object {
        @Volatile
        private var INSTANCE: MySingleton? = null
        fun getInstance(context: Context) =
            INSTANCE ?: synchronized(this) {
                INSTANCE ?: MySingleton(context).also {
                    INSTANCE = it
                }
            }
    }
    
        private val requestQueue: RequestQueue by lazy {
        // applicationContext is key, it keeps you from leaking the
        // Activity or BroadcastReceiver if someone passes one in.
        Volley.newRequestQueue(context.applicationContext)
    }
    fun <T> addToRequestQueue(req: Request<T>) {
        requestQueue.add(req)
    }
    }
    

   

