# spike_apis

First, define Enums for the filter and placement options:

```typescript
export enum FilterOptions {
  Type = 'type',
  ExtraData = 'extraData',
  Language = 'lang',
  DataKeys = 'dk',
  AgeGroup = 'ageGroup',
  Gender = 'gender',
  Longitude = 'longitude',
  Latitude = 'latitude',
  GroupPlacements = 'groupPlacements',
  Count = 'count',
}

export enum PlacementOptions {
  MainBanner = 'MainBanner',
  StripBanner = 'StripBanner',
  Products = 'Products',
  Search = 'Search',
  Brand = 'Brand',
  Popup = 'Popup',
  PushNotification = 'PushNotification',
  ContentCard = 'ContentCard',
  Other = 'Other',
  Html = 'Html',
}
```

Next, define the structure of an ad:

```typescript
export interface Ad {
  // Define the structure of an ad here, based on the data provided by the SPIKE platform.
  // This might include fields like content, targetUrl, assets, and metadata.
}
```

Then, implement the `AdsService`:

```typescript
import { HttpService, Injectable } from '@nestjs/common';
import { AxiosResponse } from 'axios';
import { Ad } from './ad.interface';
import { FilterOptions } from './filter-options.enum';
import { PlacementOptions } from './placement-options.enum';

@Injectable()
export class AdsService {
  constructor(private httpService: HttpService) {}

  async getAds(placement: PlacementOptions, filters: Record<FilterOptions, string>): Promise<Ad[]> {
    const publisherId = 'your-publisher-id'; // Replace with your publisher ID
    const key = 'your-api-access-key'; // Replace with your API access key

    // Start constructing the URL for the API request
    let url = `https://api-sandbox.spikecom.net/api/adserve/getplacement/?publisherId=${publisherId}&key=${key}&placement=${placement}`;

    // Add the filters to the URL as query parameters
    for (const filter in filters) {
      if (filters.hasOwnProperty(filter)) {
        url += `&${filter}=${filters[filter]}`;
      }
    }

    // Make the API request
    const response: AxiosResponse = await this.httpService.get(url).toPromise();

    // Check the response status
    if (response.status !== 200) {
      throw new Error('Failed to retrieve ads from the SPIKE platform');
    }

    // Transform the response data into an array of Ad objects
    const ads: Ad[] = response.data.map(adData => {
      return {
        // Fill in the details based on the structure of the response data
      };
    });

    return ads;
  }
}
```

Finally, implement the `AdsController`:

```typescript
import { Controller, Get, Param, Query, HttpException, HttpStatus } from '@nestjs/common';
import { AdsService } from './ads.service';
import { Ad } from './ad.interface';
import { FilterOptions } from './filter-options.enum';
import { PlacementOptions } from './placement-options.enum';

@Controller('ads')
export class AdsController {
  constructor(private readonly adsService: AdsService) {}

  @Get(':placement')
  async getAds(@Param('placement') placement: PlacementOptions, @Query() filters: Record<FilterOptions, string>): Promise<Ad[]> {
    try {
      const ads = await this.adsService.getAds(placement, filters);
      if (!ads.length) {
        throw new HttpException('No ads available for the specified placement', HttpStatus.NO_CONTENT);
      }
      return ads;
    } catch (error) {
      throw new HttpException(error.message, HttpStatus.INTERNAL_SERVER_ERROR);
    }
  }
}
```

This implementation includes the structure of an ad, the service for retrieving ads, and the controller for handling API requests. It also includes error handling and the use of Enums for the filter and placement options. Please note that you'll need to replace `'your-publisher-id'` and `'your-api-access-key'` with your actual publisher ID and API access key, and fill in the details of the `Ad` interface and the response data transformation based on your specific needs and the SPIKE platform's API documentation.
