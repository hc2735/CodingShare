Paste the code into the txt is ok
===========

from datetime import datetime as dt
import csv

# DataStucture Layer
######################################################
#######################################################

SecurityAttribute=['IssuerDesc','SecurityCode','SecurityDesc','CustodianCode','CurrencyCode',
                   'BbergCode', 'BloombergMarketSectorCode', 'SecurityTypeCode','SectorDesc', 
                   'GeographyRegionDesc', 'GicsSectorDesc']
StockAttribute=['ExchangeCode','Sedol','Cusip','Isin']
BondAttribute=['Maturity','Cusip','Isin','AccrualStartDate','FrequencyCode','Coupon','IssuedDate']
OptionAttribute=['Sedol','ExchangeCode','Strike','ContractSize','ExcerciseType','PutCall','Expiration']

# Abstract Class
########################################
class Security(object):
# we construct a security object for each line we read from the database
    def __init__(self,SecurityElement):
         for attribute in SecurityAttribute:
            setattr(self,attribute,SecurityElement[attribute])
#For Q3.e, we merge the objects which represent the identical security, and return the Total Number    
    @staticmethod
    def distinct_list_number(security_list):
       number = 0
       store = []
       store_list=[]
       for element in security_list:
          if element.SecurityCode not in store:
               store.append(element.SecurityCode)
               store_list.append(element)
               number = number +1           
       return [store_list, number]
#For Q3.d , we count all the security which has the same attribute value, ex."Health" Sector  
    @staticmethod
    #input the distinct list to get the number of Securities that belong to the Health Care Sector
    def Health_Sector(distinct_security_list):
        number = 0         
        for element in distinct_security_list:
            if element.SectorDesc == "Health Care":
                number = number + 1
        return number
 
# Specific Class 
# Each specific security class need to call the parent class construction using super
#######################################       
class Stock(Security):
  
    def __init__(self,SecurityElement):
         super(Stock,self).__init__(SecurityElement)
         for attribute in StockAttribute:
            setattr(self,attribute,SecurityElement[attribute]) 
                                                        
class Bond(Security):
    def __init__(self,SecurityElement):
         super(Bond,self).__init__(SecurityElement)
         for attribute in BondAttribute:
            setattr(self,attribute,SecurityElement[attribute])
         #we need to transfer the string into time type
         if self.Maturity!="NULL":
           self.Maturity=dt.strptime(self.Maturity,"%m/%d/%Y")
    #For Q3.e.(c) we need a function take the bond list output the bond with Max maturity                     
    @staticmethod        
    def Maxmaturity(bond_list):
        # we need to extract all the bond with valid maturity
        Extractbond_list=[x for x in bond_list if x.Maturity!="NULL"]        
        return max(Extractbond_list,key=lambda x:x.Maturity)
        
class Option(Security):

    def __init__(self,SecurityElement):
         super(Option,self).__init__(SecurityElement)
         for attribute in OptionAttribute:
            setattr(self,attribute,SecurityElement[attribute]) 
   #For  Q3.e.(b) function to count call option and put option
    @staticmethod
    def put_call(distinct_option_list):
        number_put = 0
        number_call = 0
        for element in distinct_option_list:
            if element.PutCall == "P":
                number_put = number_put +1
            elif element.PutCall == "C":
                number_call = number_call +1
        return [number_put,number_call] 


# Input Layer
##########################################
##########################################

fileName = r".\PortfolioAppraisal2.csv"               
class ReadtheData():
# we can define different functions to read data of various formats, following is the csv version  
    def csvRead(self,csvfile):
        try:
            positionsFile = None
            if not csv.Sniffer().has_header(csvfile):
                raise Exception("headers expected in position file. FileName: " + csvfile)
            positionsFile = open(csvfile, 'rt')
            ListofdictSecurity = csv.DictReader(positionsFile)
            return ListofdictSecurity
        except Exception, e:
            print e
            raise e
    
# After reading the data, we get the list of security dictionary, we need to construct security from the dictionary 
    def LoadSecurity(self,ListofdictSecurity):
      SecurityList=[] 
      StockList=[]
      BondList=[]
      OptionList=[]
      SecurityList=[StockList,BondList,OptionList]
      for row in ListofdictSecurity:
          if  row["SecurityTypeCode"]=="Stock":
               StockList.append(Stock(row))
          elif row["SecurityTypeCode"]=="Bond":
               BondList.append(Bond(row))
          elif row["SecurityTypeCode"]=="Option":
               OptionList.append(Option(row))
      return SecurityList


#Now Load the data into our classes, finally we get a list,
#the members of list are stock list, bond list, option list
IDoNothingbutReadData=ReadtheData()
ListofSecurityDist=IDoNothingbutReadData.csvRead(fileName)
Security_List=IDoNothingbutReadData.LoadSecurity(ListofSecurityDist)





#Application Layer
###############################
###############################


#For Q 3.e.(a) count distinct securities for each type, and merge the identical itmes
[Security_List[0],StockNum]= Stock.distinct_list_number(Security_List[0])
print "%d distinct stock"%(StockNum)
[Security_List[1],BondNum]=Bond.distinct_list_number(Security_List[1])
print "%d distinct bond"%(BondNum)
[Security_List[2],OptionNum]=Option.distinct_list_number(Security_List[2])
print "%d distinct option"%(OptionNum)


#For Q 3.(b) 
[put_number,call_number] = Option.put_call(Security_List[2])
print "There are %d put options and %d call options" %(put_number, call_number)

#For Q 3.(c)
targetBond=Bond.Maxmaturity(Security_List[1])
print  targetBond.SecurityDesc,targetBond.Maturity

#For Q 3.e.(D)
stock_health = Security.Health_Sector(Security_List[0])
bond_health = Security.Health_Sector(Security_List[1])
option_health = Security.Health_Sector(Security_List[2])
print "%d securities belong to the health care sector"% (stock_health + bond_health + option_health)