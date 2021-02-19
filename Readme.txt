Originally started as a text on Rextester, see a functional copy here: http://www.kwatech.at/CodeSamples/PROJECTS.TEST.019/LVS.PROJECTS/LV_SampleCode4UniqueFormValue().cpp
and it looks like this (a full copy) here:
///////////////// INDEV ////////////////////////////////

//Microsoft (R) C/C++ Optimizing Compiler Version 19.00.23506 for x86
//This demonstrates the using of operator overloading in order the
// accesssor of an object of a designated special-purpose class to have
// unique morph for getting and setting data (reading or writing data 
// in a memory location, external OLE document etc)
// E.g. Instead of "selection.setValue(12)" then "cout<<selection.getValue()"
// we will have "selection.value() = 12" and "cout<<selection.value()" will
// print "12" - this seems to be useful for easier to do code translator
// from VBA to C++ but uses alot the operator overloading (overhead result)
// This limited sample works; implementing it in a real project may require
// a lot of additional work, if it will be finally working anyway!
// If you don't understand why this thing is useful you don't need it.
// Code by Twareintor &mailto:ciutacu/o]gmail*com& - 2017

#include <iostream>

#define PROPERTY_SET 1
#define PROPERTY_GET 2
#define METHOD_APP 4

typedef struct _tagVariant
{ 
    // this mocks a variant from Windows
    unsigned vt; // variant type
    int intVal; // integer data
    float fltVal; // float data
    double dblVal; // double data
    char* sz_Val; // char array data
    // operator overloading for every possible data:
    void operator=(int i){intVal = i; vt = 1;}
    void operator=(float f){fltVal = f; vt = 2;}
    void operator=(double x){dblVal = x; vt = 4;}
    void operator=(char* s){sz_Val = s; vt = 8;}
} MYVARIANT;

MYVARIANT g_testValue; // this mocks a specific data stored in 
    // ... a COM object or everything else (a text/ binary file, 
    // a temporary storage, pipe, port etc)

class CDispatch
{
    // empty
    // in this example, CxDispatch does nothing; is used for fitting 
    // on the morphology of the COM application...
};

class CxDisp
{
    CDispatch* m_pDisp;
public:
    CxDisp():m_pDisp(nullptr){}
    void setDisp(CDispatch* pDisp){m_pDisp = pDisp;}
    CDispatch* getDisp(){return m_pDisp;}
    
};
unsigned MyWrap(unsigned dFlag, CxDisp* pDisp, MYVARIANT* value, ...)
{
    // a simple simulation of the wrap function that simply sets the 
    // global variable - in order the simulation to be more relevant
    if(PROPERTY_SET==dFlag)g_testValue = *value;
    else if(PROPERTY_GET==dFlag)*value = g_testValue;
    else if(METHOD_APP==dFlag) true;
    else goto ERR_FLAG;
    return 1;
ERR_FLAG:
    return 0;
}

class CValue
{
    // a mock object for attribute "Value" that will make the
    // direct contact with the user data: can be the property Value(),
    // Color(), ColorIndex(), Bold(), Italic(), ... every property 
    CxDisp* pdisp_;
protected:
    CxDisp* getDisp() const{return pdisp_;}
public:
    CValue(): pdisp_(0){};
    ~CValue(){if(pdisp_) delete pdisp_;}
    // overloading scalar assignment to MYVARIANT operator:
    template <class T> // every possible type of data...
    // void operator=(T x){g_testValue = std::forward<T&&>(x);}
    void operator=(T x)
    {
        MYVARIANT var;
        var = std::forward<T&&>(x); 
        // MYVARIANT* pvar = &var; 
        MyWrap(PROPERTY_SET, this->getDisp(), &var);
    }
    // overloading "<<" operator for the value
    friend std::ostream& operator<<(std::ostream& out, const CValue& val)
    {
        MYVARIANT var;
        MyWrap(PROPERTY_GET, val.getDisp(), &var);
        if(1==var.vt) out<<var.intVal;
        if(2==var.vt) out<<var.fltVal;
        if(4==var.vt) out<<var.dblVal;
        if(8==var.vt) out<<var.sz_Val;
        return out;
    }
    template <class T>
    bool operator==(T x)
    {
        MYVARIANT var;
        MyWrap(PROPERTY_GET, this->getDisp(), &var);
        if(1==var.vt) return x==var.intVal;
        if(2==var.vt) return x==var.fltVal;
        if(4==var.vt) return x==var.dblVal;
        // if(8==var.vt) return x==var.sz_Val; 
        // ... this last one requires specialization: see below
    }
    template <>
    bool operator==<char*>(char* x)
    {
        MYVARIANT var;
        MyWrap(PROPERTY_GET, nullptr, &var);
        if(8==var.vt) return x==var.sz_Val;
    }
};


class CItem
{
    CValue& m_value;
public:
    CItem():m_value(CValue()){}
    CValue& Value(){return m_value;}
};


int main()
{
    CItem item = CItem();
    item.Value() = 11;
    g_testValue.intVal+=25;
    std::cout<<item.Value()<<std::endl;
    item.Value() = 12.f;
    g_testValue.fltVal+=24.f;
    // std::cout<<g_testValue.vt<<std::endl;
    // std::cout<<item.Value().vt<<std::endl;
    std::cout<<item.Value()<<std::endl;
    if(item.Value()==36.f) std::cout<<"numeric stored"<<std::endl;
    char* szTest = new char[0xff];
    memset(szTest, 0, 0xff);
    memcpy(szTest, "the quick brown fox\0", 20);
    item.Value() = szTest;
    std::cout<<item.Value()<<std::endl;
    if(item.Value()==szTest)
        std::cout<<"string stored"<<std::endl;
    delete[] szTest;
    ///
    std::cout << "Hello, world!\n";
}


