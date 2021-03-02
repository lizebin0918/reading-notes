# 时间工具类-TimeUtils 

> 日常工作自己用的时间工具类，可以支持JDK1.8，不喜勿拍

```java
public final class TimeUtils {

	private TimeUtils() {
		throw new Error("Don't instance" + TimeUtils.class.getName());
	}

    /**
     * 格式为:yyyyMMddHHmmssSSS
     */
    public static final String DATE_FORMAT_yyyyMMddHHmmssSSS = "yyyyMMddHHmmssSSS";
    /**
     * 格式为:yyyy-MM-dd HH:mm:ss.SSS
     */
    public static final String DATE_FORMAT_yyyy_MM_dd_HH_mm_ss_SSS = "yyyy-MM-dd HH:mm:ss.SSS";
    /**
     * 格式为:yyyyMMddHHmmss
     */
    public static final String DATE_FORMAT_yyyyMMddHHmmss = "yyyyMMddHHmmss";
    /**
     * 格式为:yyyy-MM-dd HH:mm:ss
     */
    public static final String DATE_FORMAT_yyyy_MM_dd_HH_mm_ss = "yyyy-MM-dd HH:mm:ss";
    /**
     * 格式为:HHmmss
     */
    public static final String DATE_FORMAT_HHmmss = "HHmmss";
    /**
     * 格式为:HH:mm:ss
     */
    public static final String DATE_FORMAT_HH_mm_ss = "HH:mm:ss";
    /**
     * 格式为:yyyyMMdd
     */
    public static final String DATE_FORMAT_yyyyMMdd = "yyyyMMdd";
    /**
     * 格式为:yyyy-MM-dd
     */
    public static final String DATE_FORMAT_yyyy_MM_dd = "yyyy-MM-dd";
    /**
     * 格式为:yyyy-MM-dd HH:mm
     */
    public static final String DATE_FORMAT_yyyy_MM_dd_HH_mm = "yyyy-MM-dd HH:mm";
    /**
     * 格式为:yyyyMMddHHmm
     */
    public static final String DATE_FORMAT_yyyyMMddHHmm = "yyyyMMddHHmm";
    /**
     * 格式为:yyyy-MM-dd HH
     */
    public static final String DATE_FORMAT_yyyy_MM_dd_HH = "yyyy-MM-dd HH";
    /**
     * 格式为:yyyyMMddHH
     */
    public static final String DATE_FORMAT_yyyyMMddHH = "yyyyMMddHH";
    /**
     * 格式为:yyyy-MM
     */
    public static final String DATE_FORMAT_yyyy_MM = "yyyy-MM";
    /**
     * 格式为:yyyyMM
     */
    public static final String DATE_FORMAT_yyyyMM = "yyyyMM";
    /**
     * 格式为:HH:mm
     */
    public static final String DATE_FORMAT_HH_mm = "HH:mm";
    /**
     * 格式为:HHmm
     */
    public static final String DATE_FORMAT_HHmm = "HHmm";
    /**
     * 格式为:MM:dd
     */
    public static final String DATE_FORMAT_MM_dd = "MM:dd";
    /**
     * 格式为:MMdd
     */
    public static final String DATE_FORMAT_MMdd = "MMdd";

    private static final Map<String, DateTimeFormatter> DATE_FORMAT_MAPPING = new HashMap<String, DateTimeFormatter>(50);

    static {
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyyMMddHHmmssSSS, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyyMMddHHmmssSSS));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyyMMddHHmmss, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyyMMddHHmmss));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyy_MM_dd_HH_mm_ss, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyy_MM_dd_HH_mm_ss));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyy_MM_dd_HH_mm_ss_SSS, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyy_MM_dd_HH_mm_ss_SSS));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyyMMddHHmm, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyyMMddHHmm));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyy_MM_dd_HH_mm, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyy_MM_dd_HH_mm));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyyMMddHH, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyyMMddHH));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyy_MM_dd_HH, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyy_MM_dd_HH));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyyMMdd, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyyMMdd));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyy_MM_dd, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyy_MM_dd));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyyMM, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyyMM));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_yyyy_MM, DateTimeFormatter.ofPattern(DATE_FORMAT_yyyy_MM));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_HHmmss, DateTimeFormatter.ofPattern(DATE_FORMAT_HHmmss));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_HH_mm_ss, DateTimeFormatter.ofPattern(DATE_FORMAT_HH_mm_ss));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_HHmm, DateTimeFormatter.ofPattern(DATE_FORMAT_HHmm));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_HH_mm, DateTimeFormatter.ofPattern(DATE_FORMAT_HH_mm));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_MMdd, DateTimeFormatter.ofPattern(DATE_FORMAT_MMdd));
        DATE_FORMAT_MAPPING.put(DATE_FORMAT_MM_dd, DateTimeFormatter.ofPattern(DATE_FORMAT_MM_dd));
    }

    /*线程变量:{"dateFormat":DateFormat}*/
	private static final ThreadLocal<Map<String, SimpleDateFormat>> innerVariables = new ThreadLocal<>();
	
	private static SimpleDateFormat getDateFormatInstance(final String pattern) {
		Map<String, SimpleDateFormat> sdfMap = innerVariables.get();
		if(sdfMap == null) {
			sdfMap = new HashMap<>();
			innerVariables.set(sdfMap);
		}
        SimpleDateFormat sdf = sdfMap.get(pattern);
		if (sdf == null) {
			sdf = new SimpleDateFormat(pattern) {
                {
                    super.applyPattern(pattern);
                }
                @Override
                public void applyPattern(String pattern) {}//sdfMap存储了{"日期格式":DateFormat},所以DateFormat.applyPattern()不可用
            };
			sdfMap.put(pattern, sdf);
		}
		return sdf;
	}
	
	/**
	 * 返回当月月份前n个月份的年月
	 * 
	 * @param today
	 * @param n
	 * @return
	 */
	public static String getPreMonthReturnYearMonth(String today,
			String format, int n) {
		String result = null;
		try {
			String tmp = changeStrTimeFormat(today, format,
					DATE_FORMAT_yyyyMMdd);
			int year = Integer.parseInt(tmp.substring(0, 4));
			int month = Integer.parseInt(tmp.substring(4, 6));
			Calendar calendar = Calendar.getInstance();
			month = month - n - 1;
			if (month < 0) {
				year = year - 1;
				month = month + 12;
			}
			calendar.set(year, month, 1, 0, 0, 0);
            SimpleDateFormat sdf = getDateFormatInstance("yyyyMM");
			result = sdf.format(new Date(calendar.getTime().getTime()));
		} catch (Exception e) {
			e.printStackTrace();
		}
		return result;
	}

	/**
	 * 返回当月最后一天日期
	 * 
	 * @param today
	 * @return
	 */
	public static String getLastDayOfMonth(String today,
			String inputFormat, String outputFormat) {
		String result = null;
		try {
			String tmp = changeStrTimeFormat(today, inputFormat,
					DATE_FORMAT_yyyyMMdd);
			int year = Integer.parseInt(tmp.substring(0, 4));
			int month = Integer.parseInt(tmp.substring(4, 6));
			if (month == 12) {
				year = year + 1;
				month = 0;
			}
			Calendar calendar = Calendar.getInstance();
			calendar.set(year, month, 1, 0, 0, 0);
            SimpleDateFormat sdf = getDateFormatInstance(outputFormat);
			result = sdf.format(new Date(calendar.getTime().getTime() - 1000));
		} catch (Exception e) {
			e.printStackTrace();
		}
		return result;
	}

	/**
	 * 返回上一个月最后一天日期
	 * 
	 * @param today
	 * @return
	 */
	public static String getPreMonthLastDate(String today, String inputFormat,
			String outputFormat) {
		String result = null;
		try {
			String tmp = changeStrTimeFormat(today, inputFormat,
					DATE_FORMAT_yyyyMMdd);
			int year = Integer.parseInt(tmp.substring(0, 4));
			int month = Integer.parseInt(tmp.substring(4, 6));
			Calendar calendar = Calendar.getInstance();
			calendar.set(year, month - 1, 1, 0, 0, 0);
			SimpleDateFormat sdf = getDateFormatInstance(outputFormat);
			result = sdf.format(new Date(
					calendar.getTime().getTime() - 1000));
		} catch (Exception e) {
			e.printStackTrace();
		}
		return result;
	}

	/**
	 * 返回下一个月第一天日期
	 * 
	 * @param today
	 * @return
	 */
	public static String getNextMonthFirstDate(String today,
			String inputFormat, String outputFormat) {
		String result = null;
		try {
			String tmp = changeStrTimeFormat(today, inputFormat,
					DATE_FORMAT_yyyyMMdd);
			int year = Integer.parseInt(tmp.substring(0, 4));
			int month = Integer.parseInt(tmp.substring(4, 6));
			if (month == 12) {
				year = year + 1;
				month = 0;
			}
			Calendar calendar = Calendar.getInstance();
			calendar.set(year, month, 1, 0, 0, 0);
            SimpleDateFormat sdf = getDateFormatInstance(outputFormat);
			result = sdf.format(calendar.getTime());
		} catch (Exception e) {
			e.printStackTrace();
		}
		return result;
	}

	/**
	 * 返回本一个月第一天日期
	 * 
	 * @param today
	 * @return
	 */
	public static String getFirstDayOfMonth(String today,
			String inputFormat, String outputFormat) {
		String result = null;
		try {
			String tmp = DATE_FORMAT_yyyyMMdd.equals(inputFormat) ? today : changeStrTimeFormat(today, inputFormat, DATE_FORMAT_yyyyMMdd);
			int year = Integer.parseInt(tmp.substring(0, 4));
			int month = Integer.parseInt(tmp.substring(4, 6));
			Calendar calendar = Calendar.getInstance();
			calendar.set(year, month - 1, 1, 0, 0, 0);
            SimpleDateFormat sdf = getDateFormatInstance(outputFormat);
			result = sdf.format(calendar.getTime());
		} catch (Exception e) {
			e.printStackTrace();
		}
		return result;
	}

	/**
	 * 格式日期转换
	 * 
	 * @param date
	 * @param format
	 * @return
	 */
	public static String dateFormat(Date date, String format) {
		String result = null;
		try {
			if (date == null)
				result = "";
			else {
				SimpleDateFormat sdf = getDateFormatInstance(format);
				result = sdf.format(date);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return result;
	}

	/**
	 * 字符串日期转换成Date型日期
	 * 
	 * @param date
	 * @param format
	 * @return
	 */
	public static Date strTimeToDate(String date, String format) {
		Date result = null;
		try {
            SimpleDateFormat sdf = getDateFormatInstance(format);
			result = sdf.parse(date);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return result;
	}

	/**
	 * 字符串日期格式转换
	 * 
	 * @param date
	 * @param oldFormat
	 * @param newFormat
	 * @return
	 */
	public static String changeStrTimeFormat(String date, String oldFormat,
			String newFormat) {
		String result = null;
		try {
			if (date == null || date.equals(""))
				return "";
			else {
                SimpleDateFormat sdf = getDateFormatInstance(oldFormat);
				Date tmp = sdf.parse(date);
				sdf = getDateFormatInstance(newFormat);
				result = sdf.format(tmp);
			}
		} catch (Exception e) {
			// e.printStackTrace();
		}
		if (result == null) {
			return "";
		}
		return result;
	}

	/**
	 * 得到当前日期
	 **/
	public static String getCurDate(String dateFormat) {
        SimpleDateFormat sdf = getDateFormatInstance(dateFormat);
		Calendar c1 = Calendar.getInstance(); // today
		return sdf.format(c1.getTime());
	}

	/**
	 * 计算从date开始n天以前（以后）的日期
	 * 
	 * @param date
	 * @param dateCnt
	 * @return
	 */
	public static Date getDateRelateToDate(Date date, int dateCnt) {
		Calendar calendar = Calendar.getInstance();
		calendar.setTime(date);
		calendar.add(Calendar.DATE, dateCnt);
		return calendar.getTime();
	}
	
	/**
	 * 获取当周第一天，周一作为一个星期的第一天
	 * @param today
	 * @param inputDateFormat
	 * @param outputDateFormat
	 * @return
	 */
	public static String getFirstDayOfWeek(String today, String inputDateFormat, String outputDateFormat) {
		Date sourceDate = changeStrToDate(today, inputDateFormat);
		Calendar calendar = Calendar.getInstance();
        calendar.setFirstDayOfWeek(Calendar.MONDAY);
        calendar.setTime(sourceDate);
        calendar.set(Calendar.DAY_OF_WEEK,
                      calendar.getFirstDayOfWeek());
        return dateFormat(calendar.getTime(), outputDateFormat);
	}
	
	/**
	 * 获取当周最后一天，周日为一个星期的最后一天
	 * @param today
	 * @param inputDateFormat
	 * @param outputDateFormat
	 * @return
	 */
	public static String getLastDayOfWeek(String today, String inputDateFormat, String outputDateFormat) {
		Date sourceDate = changeStrToDate(today, inputDateFormat);
		Calendar calendar = Calendar.getInstance();
        calendar.setFirstDayOfWeek(Calendar.MONDAY);
        calendar.setTime(sourceDate);
        calendar.set(Calendar.DAY_OF_WEEK, calendar.getFirstDayOfWeek() + 6);
        calendar.add(Calendar.DATE, 1);
        return dateFormat(new Date(calendar.getTime().getTime() - 1000), outputDateFormat);
	}
	
	/**
	 * 将字符串转换为日期类型
	 * 
	 * @param date
	 * @param format
	 * @return
	 */
	public static Date changeStrToDate(String date, String format) {
        SimpleDateFormat sf = getDateFormatInstance(format);
		Date dt = null;
		try {
			dt = sf.parse(date);
		} catch (ParseException e) {
			e.printStackTrace();
		}
		return dt;
	}

	/**
	 * 检查字符串是否给定的日期格式
	 * 
	 * @param date
	 * @param format
	 * @return
	 */
	public static boolean checkDate(String date, String format) {
		if (null == date || "".equals(date.trim())) {
			return false;
		}

		try {
            SimpleDateFormat dateFormat = getDateFormatInstance(format);
			Date formatDate = dateFormat.parse(date);
			return changeStrTimeFormat(date, format, format).equals(dateFormat.format(formatDate));
		} catch (ParseException e) {
			return false;
		}
	}

	/**
	 * 获取时间date1与date2相差的秒数
	 * 
	 * @param date1
	 *            起始时间
	 * @param date2
	 *            结束时间
	 * @return 返回相差的秒数
	 */
	public static long getOffsetSeconds(Date date1, Date date2) {
		long seconds = (long) ((date2.getTime() - date1.getTime()) / 1000);
		return seconds;
	}

	/**
	 * 获取时间date1与date2相差的分钟数
	 * 
	 * @param date1
	 *            起始时间
	 * @param date2
	 *            结束时间
	 * @return 返回相差的分钟数
	 */
	public static long getOffsetMinutes(Date date1, Date date2) {
		return getOffsetSeconds(date1, date2) / 60;
	}

	/**
	 * 获取时间date1与date2相差的小时数
	 * 
	 * @param date1
	 *            起始时间
	 * @param date2
	 *            结束时间
	 * @return 返回相差的小时数
	 */
	public static long getOffsetHours(Date date1, Date date2) {
		return getOffsetMinutes(date1, date2) / 60;
	}

	/**
	 * 获取时间startDate与endDate相差的天数数(endDate - startDate)
	 * 
	 * @param startDate
	 *            起始时间
	 * @param endDate
	 *            结束时间
	 * @return 返回相差的天数
	 */
	public static long getOffsetDays(Date startDate, Date endDate) {
		return getOffsetHours(startDate, endDate) / 24;
	}

	/**
	 * 获取时间date1与date2相差的周数
	 * 
	 * @param date1
	 *            起始时间
	 * @param date2
	 *            结束时间
	 * @return 返回相差的周数
	 */
	public static long getOffsetWeeks(Date date1, Date date2) {
		return getOffsetDays(date1, date2) / 7;
	}
	
	/**
	 * 获取时间startDate与endDate相差的月数
	 * 
	 * @param startDate
	 *            起始时间
	 * @param endDate
	 *            结束时间
	 * @return 返回相差的天数
	 */
	public static int getOffsetMonths(Date startDate, Date endDate) {
		Calendar calendar = Calendar.getInstance();
		calendar.setTime(startDate);
		int curYear = calendar.get(Calendar.YEAR);
		int curMonths =  calendar.get(Calendar.MONTH) + 1;
		int curDays = calendar.get(Calendar.DAY_OF_MONTH);
		calendar.setTime(endDate);
		int endYear = calendar.get(Calendar.YEAR);
		int endMonths =  calendar.get(Calendar.MONTH) + 1;
		int endDays = calendar.get(Calendar.DAY_OF_MONTH);
		int uy = (endYear - curYear) * 12 + (endMonths - curMonths) + (endDays < curDays ? -1 : 0); 
		return uy;
	}
	
	/**
	 * 时间计算 +(-)N分钟
	 * @param date
	 * @param minute
	 * @return
	 */
	public static Date rollMinute(Date date, int minute){
		Calendar calendar = Calendar.getInstance();
		calendar.setTime(date);
		calendar.add(Calendar.MINUTE, minute);
		Date date1 = new Date(calendar.getTimeInMillis());
		return date1;
	}
	
	/**
	 * 时间计算 +(-)N小时
	 * @param date
	 * @param hour
	 * @return
	 */
	public static Date rollHour(Date date, int hour){
		Calendar calendar = Calendar.getInstance();
		calendar.setTime(date);
		calendar.add(Calendar.HOUR_OF_DAY, hour);
		Date date1 = new Date(calendar.getTimeInMillis());
		return date1;
	}
	
	/**
	 * 获取指定日期累加年月日后的时间
	 * 
	 * @param date
	 *            指定日期
	 * @param year
	 *            指定年数
	 * @param month
	 *            指定月数
	 * @param day
	 *            指定天数
	 * @return 返回累加年月日后的时间
	 */
	public static Date rollDate(Date date, int year, int month, int day) {
		Calendar cal = Calendar.getInstance();
		if (date != null) {
			cal.setTime(date);
		}
		cal.add(Calendar.YEAR, year);
		cal.add(Calendar.MONTH, month);
		cal.add(Calendar.DAY_OF_MONTH, day);
		return cal.getTime();
	}

	/**
	 * 获取指定日期累加指定月数后的时间
	 * 
	 * @param date
	 *            指定日期
	 * @param month
	 *            指定月数
	 * @return 返回累加月数后的时间
	 */
	public static Date rollMonth(Date date, int month) {
		return rollDate(date, 0, month, 0);
	}

	/**
	 * 获取指定日期累加指定天数后的时间
	 * 
	 * @param date
	 *            指定日期
	 * @param day
	 *            指定天数
	 * @return 返回累加天数后的时间
	 */
	public static Date rollDay(Date date, int day) {
		return rollDate(date, 0, 0, day);
	}

	/**
	 * 计算指定日期所在月份的天数
	 * 
	 * @param date
	 *            指定日期
	 * @return 返回所在月份的天数
	 */
	public static int getDayOfMonth(Date date) {
		Calendar cal = Calendar.getInstance();
		if (date != null) {
			cal.setTime(date);
		}
		int dayOfMonth = cal.getActualMaximum(Calendar.DATE);
		return dayOfMonth;
	}
	
	/**
	 * 将时间置为一天中最早的一秒
	 * @param today
	 * @param inputDateFormat
	 * @param outputDateFormat
	 * @return
	 */
	public static String getFirstSecondOfDay(String today, String inputDateFormat, String outputDateFormat){
		Date now = strTimeToDate(today, inputDateFormat);
		Calendar calendar = GregorianCalendar.getInstance();
		calendar.setTime(now);
		calendar.set(Calendar.HOUR_OF_DAY, 0);
		calendar.set(Calendar.MINUTE, 0);
		calendar.set(Calendar.SECOND, 0);
		calendar.set(Calendar.MILLISECOND, 0);
		return dateFormat(calendar.getTime(), outputDateFormat);
	}
	
	/**
	 * 将时间置为一天最晚的一秒
	 * @param today
	 * @param inputDateFormat
	 * @param outputDateFormat
	 * @return
	 */
	public static String getLastSecondOfDay(String today, String inputDateFormat, String outputDateFormat){
		Date now = strTimeToDate(today, inputDateFormat);
		Calendar calendar = GregorianCalendar.getInstance();
		calendar.setTime(now);
		calendar.set(Calendar.HOUR_OF_DAY, 23);
		calendar.set(Calendar.MINUTE, 59);
		calendar.set(Calendar.SECOND, 59);
		calendar.set(Calendar.MILLISECOND, 999);
		return dateFormat(calendar.getTime(), outputDateFormat);
	}
	
	/**
	 * 根据日期返回周几，周日为7
	 * @param date
	 * @param inputDateFormat
	 * @return
	 */
	public static int getWeekday(String date, String inputDateFormat) {
		Calendar c = Calendar.getInstance();
		c.setTime((strTimeToDate(date, inputDateFormat)));
		int weekday = c.get(Calendar.DAY_OF_WEEK) - 1;
		if(weekday <= 0) {
			return 7;
		}
		return weekday;
	}
	
	/**
	 * 计算两个日期之间相差的月份列表
	 * @param beginDate
	 * @param endDate
	 * @return
	 */
	public static List<String> getMonthList(Date beginDate, Date endDate, String format){
		List<String> monthList = new ArrayList<String>();
		return monthList;
	}

	/*===================补充 JDK1.8 特性===========================*/
    /**
     * 时间戳转化为时间<br/>
     * Created on : 2016-09-07 20:16
     * @author lizebin
     * @version V1.0.0
     * @param timeMillis
     * @return
     */
    public static LocalDateTime timeMillisToDate(long timeMillis) {
        ZoneId zoneId = ZoneId.systemDefault();
        return LocalDateTime.ofInstant(Instant.ofEpochMilli(timeMillis), zoneId);
    }

    /**
     * 获取时间startDatetime与endDatetime相差的偏移量(endDatetime - startDatetime)
     *
     * @param units
     *              {
     *              ChronoUnit.YEARS,
     *              ChronoUnit.MONTHS,
     *              ChronoUnit.WEEKS,
     *              ChronoUnit.DAYS,
     *              ChronoUnit.HOURS,
     *              ChronoUnit.MINUTES,
     *              ChronoUnit.SECONDS
     *              }
     * @param startDatetime 起始时间
     * @param endDatetime   结束时间
     * @return 返回相差的月数
     */
    public static long getOffsetBetweenDate(ChronoUnit units, LocalDateTime startDatetime, LocalDateTime endDatetime) {
        return units.between(startDatetime, endDatetime);
    }

    /**
     * 时间类型返回时间戳<br/>
     * Created on : 2016-09-18 15:43
     * @author lizebin
     * @version V1.0.0
     * @param value
     * @return
     */
    public static long dateToTimeMillis(Date value) {
        if(value == null) {
            return 0;
        }
        if(value.getClass() == java.util.Date.class || value.getClass() == Timestamp.class) {
            return ((Date)value).getTime();
        }
        return value.getTime();
    }

    /**
     * 时间类型返回时间戳<br/>
     * Created on : 2016-09-18 15:43
     * @author lizebin
     * @version V1.0.0
     * @param value
     * @return
     */
    public static long dateToTimeMillis(LocalDate value) {
        if(value == null) {
            return 0;
        }
        return ((LocalDate) value).atStartOfDay().atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();
    }

    /**
     * 时间类型返回时间戳<br/>
     * Created on : 2016-09-18 15:43
     * @author lizebin
     * @version V1.0.0
     * @param value
     * @return
     */
    public static long dateToTimeMillis(LocalTime value) {
        if(value == null) {
            return 0;
        }
        return ((LocalTime)value).atDate(LocalDate.of(1970, 01, 01)).atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();
    }

    /**
     * 时间类型返回时间戳<br/>
     * Created on : 2016-09-18 15:43
     * @author lizebin
     * @version V1.0.0
     * @param value
     * @return
     */
    public static long dateToTimeMillis(LocalDateTime value) {
        if(value == null) {
            return 0;
        }
        return ((LocalDateTime) value).atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();
    }

    /**
     * 格式日期转换
     *
     * @param timeMillis 时间戳
     * @param dateFormat
     * @return
     */
    public static String dateFormat(long timeMillis, String dateFormat) {
        return DATE_FORMAT_MAPPING.get(dateFormat).format(timeMillisToDate(timeMillis));
    }

    /**
     * 日期格式化<br/>
     * Created on : 2016-09-19 18:18
     * @author lizebin
     * @version V1.0.0
     * @param temporal
     * @param dateFormat
     * @return
     */
    public static String dateFormat(Temporal temporal, String dateFormat) {
        return DATE_FORMAT_MAPPING.get(dateFormat).format(temporal);
    }

    /**
     * 获取时间戳（毫秒为单位）<br/>
     * Created on : 2016-10-14 13:52
     * @author lizebin
     * @version V1.0.0
     * @return
     */
    public static long getTimeMillis() {
        return LocalDateTime.now().atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();
    }

    /**
     * 获取时间戳（秒为单位）<br/>
     * Created on : 2016-10-14 13:52
     * @author lizebin
     * @version V1.0.0
     * @return
     */
    public static long getTimeSecond() {
        return LocalDateTime.now().atZone(ZoneId.systemDefault()).toInstant().getEpochSecond();
    }

    /**
     * Date 转换成 LocalDateTime
     * @param date
     * @return
     */
    public static LocalDateTime dateToLocalDateTime(Date date) {
        return LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
    }
}
```

