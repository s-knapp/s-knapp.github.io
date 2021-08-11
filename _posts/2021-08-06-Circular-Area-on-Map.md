---
layout: post
title: "Python Code - Circular area around point on map"
date: 2021-08-06
categories: python proxy
permalink: /circle-map/
---

Context: Comparing single points (e.g. proxy data) to circular areas (from the field output from model) around each point. 
Instructions: modellats/lons - vectors of the lat/lon your model output uses.
              originlat/lon  - vectors of the lat/lon for each proxy data point, to be the center of each circle
              cradius        - the radius of the circles around each originlat/lon point
              circles        - the list which will contain your circles. circle area marked as 1's, all else NaN.
              
{% highlight python %}
    #this won't wrap at all at edges of map!

circles=[]
modellons=lon ###assumes 0-360###
modellats=lat 

originlat=lat_eocene  #circle center (data) lats
originlon=lon_eocene  #circle center (data) lons


if modellats.max() >90: #convert both model and proxy lats to -90,90
    modellats[:]-=90
if originlat.max() >90:
    originlat[:]-=90
    
cradius=300 #circle radius km 

eradius=6371 #earth radius km

def find_nearest(modelcoords, datacoord): #return index of model coord nearest to data coord
    array = np.asarray(modelcoords)
    idx = (np.abs(modelcoords - datacoord)).argmin()
    return idx
    
def coord_separation(coord):
    dist=coord[1]-coord[0]
    return dist
    
    
#this finds circular areas on land within X km of a point
for i in range(len(eocenepr_avg)):
    arr=np.zeros((len(lat),len(lon))) #domain of zeroes
    
    # r*dtheta = arc length
    degradius= cradius/eradius * 180/np.pi #get the radius of circle in degrees
    
    #TAKE ONLY THE MODEL COORDS WITHIN degradius OF ORIGIN COORDS (speeds calculation way up)
    
    lat_max= int(find_nearest(modellats,originlat[i]) + math.ceil(degradius/coord_separation(modellats)) )
    if lat_max > len(modellats)-1:
        lat_max=len(modellats)-1
    lat_min= int(find_nearest(modellats,originlat[i]) - math.ceil(degradius/coord_separation(modellats)) )
    if lat_min < -90:
        lat_min=math.ceil( modellats.min() )
    lon_max= int(find_nearest(modellons,originlon[i]) + math.ceil(degradius/coord_separation(modellons)) )
    if lon_max > len(modellons)-1:
        lon_max=len(modellons)-1
    lon_min= int(find_nearest(modellons,originlon[i]) - math.ceil(degradius/coord_separation(modellons)) )
    if lon_min < 0:
        lon_min=0
    
    #make boolean mask; true if within radius, else false
    mask=np.empty((len(modellat),len(modellon)),dtype=bool)
    mask[:,:]=False
    for la in range(lat_min,lat_max):
        for lo in range(lon_min,lon_max):
            if (modellons[lo] -originlon[i])**2 + (modellats[la]-originlat[i])**2 < degradius**2:
                mask[la,lo]=True # find all points whose distance (in degrees) is less than the radius
    arr[mask] = 1
    
    #arr = np.where(cdat[0].OCNFRAC[0,lats,lons]<0.7,arr,0) #I was filtering to only include areas over land
    arr = np.where(arr==0,np.nan,1)
    circles.append(arr)
    plt.contourf(modellons,modellats,arr,levels=[0,1],colors='none' ,hatches=['\\\\'])
    
#debugging plot
#fig,ax=plt.subplots(1)
#ax.set_aspect('equal')  #circles appear distorted if you don't make axes proportional
#ax.contourf(circles[123])
 {% endhighlight %}

