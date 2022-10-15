# Tích hợp

Cài đặt

```jsx
yarn add @teko-builder/client

# hoặc

npm install @teko-builder/client

```

## Sử dụng với React - chỉ xử lý các component tĩnh của Tempi

```jsx
import { builder, BuilderComponent } from "@teko-builder/client";
import "@teko-builder/client/dist/base.css";

const publishedDomain = 'vnshop.vn'

builder.init(publishedDomain, {
  env: "dev", // dev | stag | production
});

function App() {
  const [content, setContent] = useState();
  const [metaData, setMetaData] = useState();
  const slug = location.pathname.substring(1);
  
  useEffect(() => {
    const fetchPage = async () => {
      const { pbConfig, metaData } = await builder.getPublicPageV2({
        slug: slug || "/", 
        device: "desktop",
      });
      setContent(pbConfig);
      setMetaData(metaData);
    };
    fetchPage();
  }, []);

  return (
    <BuilderComponent
      content={content}
      configs={{
        ggSheetKey: {
          clientEmail: 'clientEmail',
          privateKey: 'privateKey',
        },
        terminalCode: 'vnshop',
        locationCode: 'HN001'
      }}
    />
  );
}

export default App;


```

## API

### builder

| function | Description | Type |
| --- | --- | --- |
| init | khởi tạo builder | function(domain: string, { env: 'dev' \| 'stag' \| 'production' \| 'internal' }) => void
| getPublicPage (deprecated) |  lấy data của page | function({ slug: string; device: 'desktop' \| 'mobile' }) => Promise\<[PublicPageResponse](#PublicPageResponse)\>
| getPublicPageV2 |  lấy data của page | function({ slug: string; device: 'desktop' \| 'mobile' }) => Promise\<[PublicPageResponseV2](#PublicPageResponseV2)\>
| getSSRInputs |  lấy các thông tin cần thiết sử dụng để khởi tạo dữ liệu SSR như products, megaMenu,... | function(pbConfig: [PageBuilderResponse](#PageBuilderResponse)) => { [key: string]: [SSRInput](#SSRInput)[] }

### BuilderComponent

| Property | Description | Type |
| --- | --- | --- |
| content | cấu hình của builder (dữ liêu pbConfig lấy từ builder.getPublicPageV2) | [PageBuilderResponse](#PageBuilderResponse)
| configs | các cấu hình thêm cho builder (terminalCode, locationCode, ...) | object
| mappingDynamicSlots | Mapping giữa các slot visualize của tempi và các component sử dụng của Ecommerce | [MappingDynamicSlots](#MappingDynamicSlots)
| ssrData | Dữ liệu phục vụ cơ chế SSR cho các component động | {[key: string] : [MappingDynamicSlots](#MappingDynamicSlots)}

### PublicPageResponseV2

| Property | Description | Type |
| --- | --- | --- |
| pbConfig | Cấu hình page (format của Tempi) | [PageBuilderResponse](#PageBuilderResponse) |
| metaData | Các thông tin thêm  của page như SEO, tracking, ... | [PageBuilderMetaData](#PageBuilderMetaData) |

### PublicPageResponse

| Property | Description | Type |
| --- | --- | --- |
| page | Thông tin page | [Page](#Page) |

### Page

| Property | Type |
| --- | --- |
| id | number |
| pbConfig | [PageBuilderResponse](#PageBuilderResponse) |
| metaTitle | string |
| metaDescription | string |
| metaImage | string |
| faviconUrl | string |


### PageBuilderResponse

| Property | Description | Type |
| --- | --- | --- |
| [key: string] | Cấu hình từng component (format của Tempi) | [PBComponent](#PBComponent) |

### PBComponent

| Property | Description | Type |
| --- | --- | --- |
| id |  id (unique) của từng component | string |
| parentId | id của component cha của component hiện tại, nếu là root thì parentId = null | string |
| children | danh sách id của các component là con của component hiện tại | string[] |
| tag | định danh của từng loại component. Ví dụ: row, col, image, best_selling_product,... | string |
| style | Style chung của component | React.CSSProperties |
| customAttributes | Cấu hình riêng của từng component | object |

### PageBuilderMetaData

| Property | Type |
| --- | --- |
| tekoTrackingWebMobile |  string |
| tekoTrackingWebDesktop |  string |
| headScript |  string |
| bodyScript |  string |
| conversionTracking | { facebookPixelId?: string; googleAnalyticsId?: string; googleTagManagerId?: string; googleAdsId?: string } |
| metaTitle | string |
| metaDescription | string |
| metaImage | string |
| faviconUrl | string |

### SSRInput

| Property | Description | Type |
| --- | --- | --- |
| slot | Loại slot cần sử dụng cơ chế SSR | [Slot](#Slot) |
| collectionSlug | slug của collection (dùng cho danh sách sản phẩm) - slot = `product_*` | string |
| menuZone | zone của menu (dùng cho mega menu) - slot = `mega_menu` | string |

### SSROutput

| Property | Description | Type |
| --- | --- | --- |
| slot | Loại slot cần sử dụng cơ chế SSR | [Slot](#Slot) |
| data | Dữ liệu đầu vào của slot sử dụng cơ chế SSR | unknown (quy định bởi bên tích hợp)

### MappingDynamicSlots
| Property | Description | Type |
| --- | --- | --- |
| [key: Slot] | slot trên tempi và component trên Ecommerce tương ứng | React.Element |

### Slot
`product_carousel_desktop` \| `product_grid_desktop` \| `product_carousel_mobile` \| `product_grid_mobile` \| `mega_menu`


## Sử dụng với Next - bao gồm cơ chế SSR bởi getServerSideProps

```jsx
import { builder, BuilderComponent } from "@teko-builder/client";
import {
  ProductCarouselDesktop,
  ProductGridDesktop,
  ProductCarouselMobile,
  ProductGridMobile,
  MegaMenu 
} from 'components'
import "@teko-builder/client/dist/base.css";

function Page({ pbConfig, metaData, ssrData }) {

  return (
    <>
      <CustomHead metaData={metaData} >
      <BuilderComponent
        content={pbConfig}
        ssrData={ssrData}
        mappingDynamicSlots={{
          product_carousel_desktop: ProductCarouselDesktop,
          product_grid_desktop: ProductGridDesktop,
          product_carousel_mobile: ProductCarouselMobile,
          product_grid_mobile: ProductGridMobile,
          mega_menu: MegaMenu
        }}
        configs={{
          ggSheetKey: {
            clientEmail: 'clientEmail',
            privateKey: 'privateKey',
          },
          terminalCode: 'vnshop',
          locationCode: 'HN001'
        }} // deprecated
      />
    </>
  );
};
```

```jsx
export async function getServerSideProps() {
  builder.init(publishedDomain, {
    env: "dev", // dev | stag | production | internal
  });

  const { pbConfig, metaData } = await builder.getPublicPageV2({
    slug: slug || "/", 
    device: "desktop",
  });

  const ssrInputs = builder.getSSRInputs(pbConfig);

  const ssrData = await ecommerceFunctionGetData(ssrInputs) // hàm này sẽ xử lý lấy data SSR, ví dụ products, megaMenu, ...

  // Pass data to the page via props
  return { props: { pbConfig, metaData, ssrData } }
}
```
