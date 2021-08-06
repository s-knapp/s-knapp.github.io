layout: post
title: "Python Code - Circular area around point on map"
date: 2021-08-06
categories: python proxy
permalink: /circle-map/

Context: Comparing single points (e.g. proxy data) to circular (or rectangular) areas (from the field output from model) around each point. 
{% highlight python %}
    datlist=list([0,1,2,3,4,5,6,8])
    lats=range(58,126) #lats over domain
    lons=list(range(270,288))+ list(range(0,73)) #lon over domain
    circles=[]

    #this finds circular areas on land within X km of a point
    for i in (datlist): #for each single point/proxy datum

        arr=np.zeros((len(lats),len(lons))) #domain of zeroes 
        originlat=lw['Lat'][datlist][i] #circle center lat
        originlon=lw["Long"][datlist][i] #circle center lon
        eradius=6371 #earth radius km
        cradius=600 #circle radius km 

        #specify different radii for some points
        if i==5: #larger radii for W african wind deposited 
            cradius=1100 

        # r*dtheta = arc length
        degradius= cradius/eradius * 180/np.pi #get the radius of circle in degrees

        if i==4: #make a rectangle for this point
            masklat = abs((cdat[0].lat[lats]-originlat)) < 6 # +/- 6 deg lat
            masklon = abs(((cdat[0].lon[lons]- 180) % 360) - 180 -originlon)<25 # +/- 25 deg lon
            arr[masklat & masklon] = 1
        else:
            mask = (((cdat[0].lon[lons]- 180) % 360) - 180 -originlon)**2 + abs(np.cos(cdat[0].lat[lats] * np.pi/180))*(cdat[0].lat[lats]-originlat)**2 < degradius**2 # find all points whose distance (in degrees) is less than the radius
            arr[mask.T] = 1
        arr = np.where(cdat[0].OCNFRAC[0,lats,lons]<0.7,arr,0) # take only the points which have some land
        arr = np.where(arr==0,np.nan,1)
        circles.append(arr)
 {% endhighlight %}

