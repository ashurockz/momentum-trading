# momentum-trading
#algorithm for momentum trading

#importing libraries
import pandas as pd
import numpy as np
%matplotlib inline
import matplotlib.pyplot as plt
plt.rcParams['figure.figsize']=(20.0,10.0)
from sklearn.linear_model import LinearRegression
reg = LinearRegression()

df = pd.read_csv("file.csv",header = 0,index_col="Company")
df.head()

m=[]
m.append(100000)
final = 100000                                   #amount in our account
for z in range(46):                              #we will trade for 46 months which means approx 3 and half years
    rsq =[]                                      #for storing R-square values for all NIFTY 50 stocks
    slope=[]                                     #for storing slope values for all NIFTY 50 stocks
    for c in range(50) :
        y= df.iloc[c,1 + 4*z :25 + 4*z].values   #storing weekly values of share in the training peeriod of 24 weeks( 6 months) 
        x=np.arange(24)
        reg.fit(y.reshape(-1, 1), x)
        rsq.append(reg.score(y.reshape(-1, 1), x))
        slope.append(float(reg.coef_) 
    rindex = []           #for storing index of company which comes under top 20 R-square values
    rslope = []           #for storing slope of company which comes under top 20 R-square values
    rrsq = []             #for storing R-square value of company which comes under top 20 R-square values
    mode_rslope =[]       #for storing mode of slope of company which comes under top 20 R-square values 
    for i in range(20):
        idx = np.argmax(rsq)
        rindex.append(idx)
        rslope.append(slope[idx])
        rrsq.append(rsq[idx])
        if (slope[idx] < 0) :
            mode_rslope.append(-slope[idx])
        else :
            mode_rslope.append(slope[idx])
        rsq[idx]=0
    
    pindex = []     #for storing index of company which comes under top 10 slopes(modulous) and top 5 positive slopes among previously selected companies
    pslope = []     #for storing slope of company which comes under top 10 slopes(modulous) and top 5 positive slopes among previously selected companies
    nindex = []     #for storing index of company which comes under top 10 slopes(modulous) and top 5 negative slopes among previously selected companies
    nslope = []     #for storing slope of company which comes under top 10 slopes(modulous) and top 5 negative slopes among previously selected companies
    for i in range(10):
        idx = np.argmax(mode_rslope)
        if( (rslope[idx] > 0) & (len(pindex) < 5) ):
            pindex.append(rindex[idx])
            pslope.append(rslope[idx])
        elif( (rslope[idx] < 0) & (len(nindex) < 5)):
            nindex.append(rindex[idx])
            nslope.append(rslope[idx])
        mode_rslope[idx]=0
        
    pfinal = 0  #for storing final amount after complete trading on stocks which we bought first
    nfinal = 0  #for storing final amount after complete trading on stocks which we sold first
    for i in range(len(pindex)):                                                                #here length denotes the no of stocks which are going to sell
        amount = final/(sum(pslope)-sum(nslope))                                                #we are dividing the amount in the ratio of slopes
        amount = pslope[i]*amount*(df.iloc[pindex[i],28 + 4*z]/df.iloc[pindex[i],25+4*z])
        pfinal = pfinal + amount
    for i in range(len(nindex)):                                                                #here length denotes the no of stocks which are going to buy
        amount = final/(sum(pslope)-sum(nslope))                                                #we are dividing the amount in the ratio of slopes
        amount = (-nslope[i])*amount*(df.iloc[nindex[i],25 + 4*z]/df.iloc[nindex[i],28 + 4*z])
        nfinal = nfinal + amount
    final = pfinal + nfinal
    m.append(final)
    #print(str(z+1)+" month " +"  --   After " + str(4*(z+1)) +" weeks trade final amount is : " + str(final))
    if((z%12) == 11):
        #print(m[z+1])
        #print(m[z-11])
        print("return after" + str((z+1)/12) +"year    " +str(((m[z+1]/m[z-11])-1)*100))
print(final)
