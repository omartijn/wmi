WMI
===

Created by Thomas Sparber 2016

GOAL
----
This is a very simple library written in C++ to execute WMI queries.
The aim is to make it as simple as possible and stick as much as
possible to the C++ standard (avoid Microsoft specific things) so that
it even compiles smoothly on MinGW.

Usage
-----
I think it is so simple that a small program explains the usage without
any further comments:

    int main(int /*argc*/, char */*args*/[])
    {
        try {
            Win32_ComputerSystem computer = retrieveWmi<Win32_ComputerSystem>();
            cout<<"Computername: "<<computer.Name<<"."<<computer.Domain<<endl;
            cout<<endl;
            cout<<"Installed services:"<<endl;
            for(const Win32_Service &service : retrieveAllWmi<Win32_Service>())
            {
                cout<<service.Caption<<endl;
            }
        } catch (const WmiException &ex) {
            cerr<<"Wmi error: "<<ex.errorMessage<<", Code: "<<ex.hexErrorCode()<<endl;
            return 1;
        }
    
        return 0;
    }

The include file <wmi.hpp> contains all interfaces to execute WMI queries.
The include file <wmiclasses.hpp> contains some predefined WMI classes
(e.g. Win32_ComputerSystem or Win32_Service...)

Create WMI classes
------------------
As already mentioned, the include file <wmiclasses.hpp> provides some standard
WMI classes, but it is also very easy to add more of them. All you need to do is:

    struct Win32_MyCustomClass
    {
    
        /**
          * This function is called by requestWmi and requestAllWmi
          * with the actual WmiResult
         **/
        void setProperties(const WmiResult &result, std::size_t index)
        {
		    //This macro reads all the properties from the WmiResult
			//and stores them in the given variables
            SET_VARIABLES(result, index, (*this),
                name, //and all the other WMI properties
            );
        }
    
        /**
          * This function is used to determine the actual WMI class name
         **/
        static std::string getWmiClassName()
        {
            return "Win32_MyCustomClass";
        }
    
        string name;
        //All the other properties you wish to read from WMI
    
    }; //end struct Win32_ComputerSystem

These two functions are the only requirements your class needs to have.