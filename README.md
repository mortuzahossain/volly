#  Using of _Volley_


## Add the volley library to the project

open build.gradle and add the dependency

    compile 'com.android.volley:volley:1.0.0'


## Now create a appcontroler.java file and past the code given bellow..


```java

package com.wordpress.mortuzabaust.spamsender;

/**
 * Created by Mim on 02-02-17.
 */
import android.app.Application;
import android.text.TextUtils;

import com.android.volley.Request;
import com.android.volley.RequestQueue;
import com.android.volley.toolbox.Volley;

public class AppController extends Application {

    public static final String TAG = AppController.class.getSimpleName();

    private RequestQueue mRequestQueue;

    private static AppController mInstance;

    @Override
    public void onCreate() {
        super.onCreate();
        mInstance = this;
    }

    public static synchronized AppController getInstance() {
        return mInstance;
    }

    public RequestQueue getRequestQueue() {
        if (mRequestQueue == null) {
            mRequestQueue = Volley.newRequestQueue(getApplicationContext());
        }

        return mRequestQueue;
    }

    public <T> void addToRequestQueue(Request<T> req, String tag) {
        req.setTag(TextUtils.isEmpty(tag) ? TAG : tag);
        getRequestQueue().add(req);
    }

    public <T> void addToRequestQueue(Request<T> req) {
        req.setTag(TAG);
        getRequestQueue().add(req);
    }

    public void cancelPendingRequests(Object tag) {
        if (mRequestQueue != null) {
            mRequestQueue.cancelAll(tag);
        }
    }
} ```


## In the manifest file give
	1. Internet permission and 
	2. in the <application> android:name=".AppController"</application>

## To access the data from mainactivity we have to write this type of 


```java

package com.wordpress.mortuzabaust.spamsender;

import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.TextView;

import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.VolleyLog;
import com.android.volley.toolbox.JsonArrayRequest;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

public class MainActivity extends AppCompatActivity {
    TextView showData;
    String myUrl = "https://userfun.000webhostapp.com/spamsender/api.php";
    String t1, t2 = "";
    ListView lv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        lv = (ListView) findViewById(R.id.mylist);
        fatchingData();
    }

    void fatchingData() {
        JsonArrayRequest jsonArrayRequest = new JsonArrayRequest(myUrl, new Response.Listener<JSONArray>() {
            @Override
            public void onResponse(JSONArray response) {
                final String[] from = new String[response.length()];
                final String[] to = new String[response.length()];
                final String[] gip = new String[response.length()];
                final String[] message = new String[response.length()];

                for (int i = 0; i < response.length(); i++) {

                    try {

                        JSONObject jsonObject = (JSONObject) response.get(i);
                        from[i] = jsonObject.getString("from_address");
                        to[i] = jsonObject.getString("to_address");
                        gip[i] = jsonObject.getString("ip");
                        message[i] = jsonObject.getString("message");


                    } catch (JSONException e) {
                        e.printStackTrace();
                    }


                }

                lv.setAdapter(new ArrayAdapter(getApplicationContext(), R.layout.mylistview, R.id.fromAddressinList, from));

                lv.setOnItemClickListener(new AdapterView.OnItemClickListener() {
                    @Override
                    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                        Intent intent = new Intent(MainActivity.this, Details.class);
                        intent.putExtra("from", from[position]);
                        intent.putExtra("to", to[position]);
                        intent.putExtra("ip", gip[position]);
                        intent.putExtra("message", message[position]);
                        startActivity(intent);

                    }
                });

            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                VolleyLog.d("Something Wrong");
            }
        });

        com.wordpress.mortuzabaust.spamsender.AppController.getInstance().addToRequestQueue(jsonArrayRequest);
    }
}


```

## For Details Activity

```java

package com.wordpress.mortuzabaust.spamsender;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.TextView;

public class Details extends AppCompatActivity {


    TextView from, to, ip, message;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_details);

        from = (TextView) findViewById(R.id.fromaddress);
        to = (TextView) findViewById(R.id.toaddress);
        ip = (TextView) findViewById(R.id.ip);
        message = (TextView) findViewById(R.id.message);


        String _from = getIntent().getStringExtra("from");
        String _to = getIntent().getStringExtra("to");
        String _ip = getIntent().getStringExtra("ip");
        String _message = getIntent().getStringExtra("message");

        from.setText(_from);
        to.setText(_to + "\n");
        ip.setText(_ip + "\n\n");
        message.setText(_message);
    }
}

```


# For The XML FILE THE CODE ARE LIKE THIS

## activity_main

```xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <ListView
        android:id="@+id/mylist"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="10" />

</LinearLayout>


```

## mylistview


```xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/colorAccent"
    android:orientation="vertical">


    <TextView
        android:id="@+id/fromAddressinList"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="8dp"
        android:text="dummu"
        android:textColor="#fff"
        android:textSize="18sp" />

</LinearLayout>

```

## activity_details

```xml

<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="10dp">


        <TextView
            android:id="@+id/fromaddress"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textColor="@color/colorPrimary"
            android:textSize="20dp" />

        <TextView
            android:id="@+id/toaddress"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textColor="#000"
            android:textSize="20dp" />

        <TextView
            android:id="@+id/ip"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textColor="@color/colorAccent"
            android:textSize="20dp" />

        <TextView
            android:id="@+id/message"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textColor="#000"
            android:textSize="20dp" />


    </LinearLayout>

</ScrollView>


```
