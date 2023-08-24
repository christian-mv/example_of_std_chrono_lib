# example_of_std_chrono_lib
This is just a quick example on how to use the std::chrono library in C++20. 


```cpp
// compile using: C++20 or later
#include <iostream>
#include <chrono>
#include <format>


/// shows how to print timestamp in UTC and in a specified time zone.
void formatingInUtcAndDifferentTimeZone(long long msSinceUtcEpoch) {
    using namespace std::chrono;
    //    using namespace std::literals;

    std::cout << "------ Demo: formatingInUtcAndDifferentTimeZone ----- " << std::endl;

    auto timePoint = system_clock::time_point{ milliseconds{ msSinceUtcEpoch } };
    auto as_utc = zoned_time{ locate_zone("UTC"), timePoint };
    auto as_perth = zoned_time{ locate_zone("Australia/Perth"), timePoint };


    std::cout << "timePoint: "<< timePoint << " | " << std::format("{:%X %Z}", timePoint)  << std::endl;
    std::cout << "as_utc: "<< as_utc << " | " << std::format("{:%X %Z}", as_utc) << " | " << as_utc.get_time_zone()->name() << std::endl;
    std::cout << "as_perth: "<< as_perth << " | " << std::format("{:%X %Z}", as_perth) << " | " << as_perth.get_time_zone()->name() << std::endl;
    std::cout << std::endl;
}

/// Extracts information from date and time as UTC
void extractingDateAndTimeInfo(long long msSinceUtcEpoch){
    using namespace std::chrono;
    //    using namespace std::literals;

    std::cout << "------ Demo: extractingDateAndTimeInfo  ----- " << std::endl;

    auto tp = zoned_time{current_zone(), system_clock::now()}.get_local_time();
    auto dp = floor<days>(tp);
    year_month_day ymd{dp};
    hh_mm_ss time{floor<milliseconds>(tp-dp)};

    std::cout << "Year: "<< int(ymd.year()) << std::endl;
    std::cout << "Month: "<< unsigned{ymd.month()} << std::endl;
    std::cout << "Day: "<< unsigned{ymd.day()} << std::endl;
    std::cout << "Hour: "<< time.hours().count() << std::endl;
    std::cout << "Minute: "<< time.minutes().count() << std::endl;
    std::cout << "Seconds: "<< time.seconds().count() << std::endl;
    std::cout << "Milliseconds: "<< time.subseconds().count() << std::endl;

    // alternatively, you can get the number of milliseconds since midnight as follows
    auto int_ms = duration_cast<milliseconds>(tp-dp); // number of milliseconds since midnight
    std::cout << "Total milliseconds since midnight: "<< int_ms.count() <<"\n"<< std::endl;

}

/// @returns true if a iana id is valid or not.
/// @note Im sorry if this is not the best approach.
bool is_valid_iana_id(const std::string& iana_id) {
    try {
        auto zone = std::chrono::locate_zone(iana_id);
        (void)zone; // like Q_UNUSED
        return true;
    } catch (const std::exception&) {
        return false;
    }
}

/// Extracts information from date and time into a different time zone
void extractingDateAndTimeInfoToDifferentTimeZone(long long msSinceUtcEpoch, const std::string& ianaId) {
    using namespace std::chrono;
    std::cout << "------ Demo: extractingDateAndTimeInfoToDifferentTimeZone ----- " << std::endl;

    // check tha the timezone id is valid
    if(!is_valid_iana_id(ianaId)){
        std::cout<<"Invalid iana id: "<<ianaId<<std::endl;
        return;
    }

    auto zone_ptr = std::chrono::locate_zone(ianaId);
    if (!zone_ptr) {
        std::cout<<"Impossible to extract time zone: "<<ianaId<<std::endl;
        return;
    }

    auto utcTimePoint = system_clock::time_point{ std::chrono::milliseconds{ msSinceUtcEpoch } };
    auto myZonedTime = zoned_time{ std::chrono::locate_zone(ianaId), utcTimePoint }.get_local_time();

    auto dp = floor<days>(myZonedTime);
    year_month_day ymd{dp};
    hh_mm_ss time{floor<milliseconds>(myZonedTime-dp)};

    std::cout << "Year: "<< int(ymd.year()) << std::endl;
    std::cout << "Month: "<< unsigned{ymd.month()} << std::endl;
    std::cout << "Day: "<< unsigned{ymd.day()} << std::endl;
    std::cout << "Hour: "<< time.hours().count() << std::endl;
    std::cout << "Minute: "<< time.minutes().count() << std::endl;
    std::cout << "Seconds: "<< time.seconds().count() << std::endl;
    std::cout << "Milliseconds: "<< time.subseconds().count() << std::endl;

    // alternatively, you can get the number of milliseconds since midnight as follows
    auto int_ms = duration_cast<milliseconds>(myZonedTime-dp); // number of milliseconds since midnight
    std::cout << "Total milliseconds since midnight: "<< int_ms.count() <<"\n"<< std::endl;

}

/// Shows how to create a time_point manually.
std::chrono::system_clock::time_point createTimePointsWithDateTimeParts(int w_year, int w_month, int w_day,
                                                                        int w_hours, int w_minutes, int w_seconds, int w_milliseconds){

    //https://stackoverflow.com/questions/31284994/most-elegant-way-to-combine-chronotime-point-from-hours-minutes-seconds-etc

    using namespace std::chrono;

    std::cout << "------ Demo: createTimePointsWithDateTimeParts ----- " << std::endl;

    system_clock::time_point dateTime =
        sys_days(year(w_year)
                 /month(w_month)
                 /day(w_day))
        + hours(w_hours)
        + minutes(w_minutes)
        + seconds(w_seconds)
        + milliseconds(w_milliseconds);

    std::cout << dateTime << '\n'<<std::endl;

    return dateTime;
}

/// extract the number of milliseconds since EPOCH from a given chrono time-point
long long fromTimePointToMsSinceEpoch(const std::chrono::system_clock::time_point &tp){
    std::cout << "------ Demo: fromTimePointToMsSinceEpoch ----- " << std::endl;
    auto msSinceEpoch = std::chrono::duration_cast<std::chrono::milliseconds>( tp.time_since_epoch() ).count();
    std::cout << tp<<"\tepoch: "<<msSinceEpoch<< '\n'<<std::endl;
    return msSinceEpoch;
}

/// convert from UTC milliseconds since epoch to a chrono time-point
std::chrono::system_clock::time_point fromMillisencondsSinceEpochToTimePoint(const long long msSinceEpoch){
    std::cout << "------ Demo: fromMillisencondsSinceEpochToTimePoint ----- " << std::endl;
    auto utcTimePoint = std::chrono::system_clock::time_point{ std::chrono::milliseconds{ msSinceEpoch } };
    std::cout << utcTimePoint<<"\tepoch: "<<msSinceEpoch<< '\n'<<std::endl;
    return utcTimePoint;
}

/// Print the whole IANA database
void printIanaDatabase(){
    std::cout << "------ Demo: printIanaDatabase ----- " << std::endl;

    const auto& timeZoneDatabase = std::chrono::get_tzdb(); // initialize the time zone database

    std::cout<<"IANA database version: "<<timeZoneDatabase.version<<std::endl;
    std::cout<<"current zone: "<<timeZoneDatabase.current_zone()->name()<<std::endl;
    std::cout<<"NAME\t --> TARGET: "<<std::endl;
    for(const auto &l: timeZoneDatabase.links){
        std::cout<<l.name()<<"\t --> "<<l.target()<<std::endl;
    }
}

/// @brief Obtains information about this time zone at the time point tp
/// @note this method assumes that ianaId is a valid id, otherwise undefined behaviour
void printTimeZoneAtTimePoint(const std::chrono::system_clock::time_point &utc_tp, const std::string &ianaId){

    // check tha the timezone id is valid
    std::cout << "------ Demo: printTimeZoneAtTimePoint ----- " << std::endl;

    auto zone_ptr = std::chrono::locate_zone(ianaId);

    auto info = zone_ptr->get_info(utc_tp); // check: https://learn.microsoft.com/en-us/cpp/standard-library/sys-info-struct?view=msvc-170


    // The abbreviation used for the associated time_zone and time_point.
    std::cout<<"info.abbrev: "<<info.abbrev<<std::endl;

    // Provides the range over the associated time zone, [begin, end), that the offset and abbrev apply to.
    std::cout<<"info.begin: "<<info.begin<<" UTC"<<std::endl;
    std::cout<<"info.end: "<<info.end<<" UTC"<<std::endl; // UTC

    // The Universal Time Coordinated (UTC) offset in effect for the associated time_zone and time_point.
    std::cout<<"info.offset: "<<info.offset<<std::endl;

    // Indicates whether the sys_info is on daylight savings time, and if so,
    // suggests the offset this time zone might use if it weren't on daylight savings time.
    std::cout<<"info.save: "<<info.save<<std::endl;

    std::cout<<"is DST: "<<(info.save.count()!=0?"true":"false")<<std::endl;

}

/// Returns the utc time-points when there is a DST transition
/// @note make sure that utc_tp1<=utc_tp2
std::vector<std::chrono::system_clock::time_point> getDstTransitions(const std::chrono::system_clock::time_point &utc_tp1,
                                                                     const std::chrono::system_clock::time_point &utc_tp2,
                                                                     const std::string &ianaId)
{
    std::cout << "------ Demo: getDstTransitions ----- " << std::endl;

    std::vector<std::chrono::system_clock::time_point> transitionsVect;

    // check tha the timezone id is valid
    if(!is_valid_iana_id(ianaId)){
        std::cout<<"Invalid iana id: "<<ianaId<<std::endl;
        return transitionsVect;
    }

    // check that range is ok
    if(utc_tp1>=utc_tp2){
        std::cout<<"Invalid range"<<std::endl;
        return transitionsVect;
    }

    auto zone_ptr = std::chrono::locate_zone(ianaId);


    auto initialTp = std::chrono::system_clock::time_point{ zone_ptr->get_info(utc_tp1).end };

    if(initialTp>utc_tp2)
        return transitionsVect;

    transitionsVect.push_back(initialTp);

    while(true){
        const auto &currentEnd = transitionsVect[transitionsVect.size()-1];
        const auto &newEnd = std::chrono::system_clock::time_point{ zone_ptr->get_info(currentEnd).end };
        if(newEnd>=utc_tp2)
            break;
        transitionsVect.push_back( newEnd );
    }

    // print
    std::cout<<"DST transitions between "<<utc_tp1<<" UTC and "<<utc_tp2<<" UTC"<<std::endl;
    for(const auto& tp: transitionsVect){
        auto zonedTime = std::chrono::zoned_time{ std::chrono::locate_zone(ianaId), tp }.get_local_time();
        std::cout<<tp<<" UTC\t->\t"<<zonedTime<<" "+ianaId<<std::endl;
    }

    return transitionsVect;
}


int main(){

    auto timePoint = createTimePointsWithDateTimeParts(2023, 10, 1, 2, 30, 0, 10);
    auto msSinceUtcEpoch = fromTimePointToMsSinceEpoch(timePoint); // timestamp given in UTC milliseconds since epoch
    auto timePoint_clone = fromMillisencondsSinceEpochToTimePoint(msSinceUtcEpoch); // convert back

    std::cout<<"timePoint_clone == timePoint: "<<(timePoint_clone==timePoint?"true\n":"false\n")<<std::endl;

    formatingInUtcAndDifferentTimeZone(msSinceUtcEpoch);

    extractingDateAndTimeInfo(msSinceUtcEpoch);

    extractingDateAndTimeInfoToDifferentTimeZone(msSinceUtcEpoch, "Australia/Melbourne");

    //    printIanaDatabase(); // print the whole database

    printTimeZoneAtTimePoint(timePoint, "Australia/Melbourne");

    auto dstTranstionsVec = getDstTransitions(createTimePointsWithDateTimeParts(2022, 10, 1, 2, 30, 0, 0),
                                              createTimePointsWithDateTimeParts(2023, 10, 1, 2, 30, 0, 0),
                                              "Australia/Melbourne");

    return 0;
}


```
