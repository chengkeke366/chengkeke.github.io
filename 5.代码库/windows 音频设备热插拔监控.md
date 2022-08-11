  **win 参考：https://zhuanlan.zhihu.com/p/442589263**

​        视频设备注册： RegisterDeviceNotification  

​        音频设备：    IMMNotificationClient与IMMDeviceEnumerator 结合来实现

```c++
#define SAFE_RELEASE(punk)  \
 if ((punk) != NULL)  \
 { (punk)->Release(); (punk) = NULL; }   

#include <windows.h>
#include <setupapi.h>  
#include <initguid.h>
#include <mmdeviceapi.h>  
#include <Functiondiscoverykeys_devpkey.h>
#include "iostream"
#include <stdio.h>
using namespace std;

class CMMNotificationClient : public IMMNotificationClient
{
public:
  IMMDeviceEnumerator* m_pEnumerator;
  CMMNotificationClient() :
    _cRef(1),
    m_pEnumerator(NULL)
  {
    // 初始化COM  
    ::CoInitialize(NULL);
    HRESULT hr = S_OK;

    // 创建接口
    hr = CoCreateInstance(
      __uuidof(MMDeviceEnumerator), NULL,
      CLSCTX_ALL, __uuidof(IMMDeviceEnumerator),
      (void**)&m_pEnumerator);

    if (hr == S_OK)
    {
      cout << "接口创建成功" << endl;
    }
    else
    {
      cout << "接口创建失败" << endl;
    }

    // 注册事件
    hr = m_pEnumerator->RegisterEndpointNotificationCallback((IMMNotificationClient*)this);
    if (hr == S_OK)
    {
      cout << "注册成功" << endl;
    }
    else
    {
      cout << "注册失败" << endl;
    }
  }

  ~CMMNotificationClient()
  {
    SAFE_RELEASE(m_pEnumerator)
      ::CoUninitialize();
  }

  // IUnknown methods -- AddRef, Release, and QueryInterface   
private:
  LONG _cRef;

  // Private function to print device-friendly name
  HRESULT _PrintDeviceName(LPCWSTR  pwstrId);

  ULONG STDMETHODCALLTYPE AddRef()
  {
    return InterlockedIncrement(&_cRef);
  }

  ULONG STDMETHODCALLTYPE Release()
  {
    ULONG ulRef = InterlockedDecrement(&_cRef);
    if (0 == ulRef)
    {
      delete this;
    }
    return ulRef;
  }

  HRESULT STDMETHODCALLTYPE QueryInterface(
    REFIID riid, VOID** ppvInterface)
  {
    if (IID_IUnknown == riid)
    {
      AddRef();
      *ppvInterface = (IUnknown*)this;
    }
    else if (__uuidof(IMMNotificationClient) == riid)
    {
      AddRef();
      *ppvInterface = (IMMNotificationClient*)this;
    }
    else
    {
      *ppvInterface = NULL;
      return E_NOINTERFACE;
    }
    return S_OK;
  }

  HRESULT STDMETHODCALLTYPE OnDefaultDeviceChanged(
    EDataFlow flow, ERole role,
    LPCWSTR pwstrDeviceId)
  {
    cout << "OnDefaultDeviceChanged:"<<pwstrDeviceId  << endl;
    return S_OK;
  }

  HRESULT STDMETHODCALLTYPE OnDeviceAdded(LPCWSTR pwstrDeviceId)
  {
    cout << "OnDeviceAdded:" <<  pwstrDeviceId <<endl;
    return S_OK;
  };

  HRESULT STDMETHODCALLTYPE OnDeviceRemoved(LPCWSTR pwstrDeviceId)
  {
    cout << "OnDeviceRemoved:" << pwstrDeviceId << endl;
    return S_OK;
  }

  HRESULT STDMETHODCALLTYPE OnDeviceStateChanged(
    LPCWSTR pwstrDeviceId,
    DWORD dwNewState)
  {
//   #define DEVICE_STATE_ACTIVE      0x00000001          插入设备
//   #define DEVICE_STATE_DISABLED    0x00000002          禁用设备 
//   #define DEVICE_STATE_NOTPRESENT  0x00000004          拔出设备 
//   #define DEVICE_STATE_UNPLUGGED   0x00000008          卸载设备
    //DEVICE_STATE_DISABLED    DEVICE_STATE_NOTPRESENT   DEVICE_STATE_UNPLUGGED 均为表示设备不可用（分别是 禁用设备，拔出设备，卸载设备）
    cout << "OnDeviceStateChanged:"<<  _PrintDeviceName(pwstrDeviceId) <<" state : "<< dwNewState << endl;
    return S_OK;
  }

  HRESULT STDMETHODCALLTYPE OnPropertyValueChanged(
    LPCWSTR pwstrDeviceId,
    const PROPERTYKEY key)
  {
    cout << "OnPropertyValueChanged" << endl;
   // _PrintDeviceName(pwstrDeviceId);
    return S_OK;
  }
};

// Given an endpoint ID string, print the friendly device name.
HRESULT CMMNotificationClient::_PrintDeviceName(LPCWSTR pwstrId)
{
  HRESULT hr = S_OK;
  IMMDevice* pDevice = NULL;
  IPropertyStore* pProps = NULL;
  PROPVARIANT varString;

  CoInitialize(NULL);
  PropVariantInit(&varString);

  if (m_pEnumerator == NULL)
  {
    // Get enumerator for audio endpoint devices.
    hr = CoCreateInstance(__uuidof(MMDeviceEnumerator),
      NULL, CLSCTX_INPROC_SERVER,
      __uuidof(IMMDeviceEnumerator),
      (void**)&m_pEnumerator);
  }
  if (hr == S_OK)
  {
    hr = m_pEnumerator->GetDevice(pwstrId, &pDevice);
  }
  if (hr == S_OK)
  {
    hr = pDevice->OpenPropertyStore(STGM_READ, &pProps);
  }
  if (hr == S_OK)
  {
    // Get the endpoint device's friendly-name property.
    hr = pProps->GetValue(PKEY_Device_FriendlyName, &varString);
  }
  printf("----------------------\nDevice name: \"%S\"\n"
    "  Endpoint ID string: \"%S\"\n",
    (hr == S_OK) ? varString.pwszVal : L"null device",
    (pwstrId != NULL) ? pwstrId : L"null ID");

  PropVariantClear(&varString);

  SAFE_RELEASE(pProps)
    SAFE_RELEASE(pDevice)
    return hr;
}

int main(int argc, TCHAR* argv[], TCHAR* envp[])
{
  CMMNotificationClient mmClient;

  cin.get();
  return 0;
}
```

