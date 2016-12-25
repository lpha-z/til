# C++のconstexprで長大なコンパイル時計算を行う方法

## clang
### clang on ubuntu
- メモリを食いつぶしてしまう

◎2万要素のソートが16GBマシンでの限界くらい

### clang on cygwin
- なぜかエラー```g++: エラー: unrecognized command line option ‘-fconstexpr-steps=1000000000’```
- llvm-bitcodeにすればいいかと思ったら```lli```で実行時に```LLVM ERROR: Program used external function '__dso_handle' which could not be resolved!```のエラーが出る
- ```clang++ -std=c++11 -fconstexpr-steps=1000000000 -c -emit-llvm -fno-use-cxa-atexit```とすることで解決できた
- 普通に```-c```でオブジェクトファイルを出力してしてからリンクをしてもいける

◎50万要素のソート：10分
### clang with Microsoft codegen on VisualStudio2015
- 普通にメモリをあまり使わずコンパイルできる

◎10万要素のソート：20分


## テストコード
```main.cpp
#include "lfl.hpp"
#include <iostream>
#include <array>

constexpr std::uint_least32_t random_w( std::size_t seed ) {
        return ( ( seed * 12345678901 >> 5 ) ^ ( seed * 98765432109 >> 5 ) ^ seed ) % 10000;
}

template<class T>
struct Less
{
        constexpr bool operator()( T a, T b ) const { return a < b; }
};

int main() {
        constexpr auto a = lfl::generate<SIZE>( random_w );
        constexpr auto b = lfl::sort( a, Less<uint_least32_t>{} );
        std::cout  << " " << b[10] << std::endl;
        return 0;
}
```
```lfl.hpp
#ifndef LFL_LFL_HPP
#define LFL_LFL_HPP
#include <cstddef>
#include <utility>

namespace lfl
{
        // index_t
        using index_t = std::ptrdiff_t;

        // index_tuple
        template<lfl::index_t... Index>
        struct index_tuple { };

        namespace detail
        {
                // index_tuple_build
                template<class IndexTuple, lfl::index_t Step, bool Even>
                struct index_tuple_build;

                template<lfl::index_t... Index, lfl::index_t Step>
                struct index_tuple_build<lfl::index_tuple<Index...>, Step, true>
                {
                        using type = lfl::index_tuple<Index..., ( Index + Step )...>;
                };

                template<lfl::index_t... Index, lfl::index_t Step>
                struct index_tuple_build<lfl::index_tuple<Index...>, Step, false>
                {
                        using type = lfl::index_tuple<Index..., ( Index + Step )..., Step * 2>;
                };

                // index_tuple_impl
                template<std::size_t N>
                struct index_tuple_impl : public lfl::detail::index_tuple_build<typename lfl::detail::index_tuple_impl<N / 2>::type, N/2, N % 2 == 0> { };

                template<>
                struct index_tuple_impl<0>
                {
                        using type = lfl::index_tuple<>;
                };

        } // detail

        // make_index_tuple
        template<std::size_t N>
        using make_index_tuple = typename lfl::detail::index_tuple_impl<N>::type;

        // s_array
        template<class T, std::size_t N>
        struct s_array
        {
                T elems[N];
                constexpr const T& operator[]( std::size_t n ) const { return elems[n]; }
                using const_reference_rawarray = const T(&)[N];
                constexpr operator const_reference_rawarray() const { return elems; }
        };

        namespace detail
        {
                // generate_impl
                template<std::size_t N, class Func, lfl::index_t... Index,
                        class RetType = lfl::s_array<decltype( std::declval<Func>()( std::declval<lfl::index_t>() ) ), N>>
                        constexpr RetType generate_impl( const Func& func, lfl::index_tuple<Index...> ) {
                        return RetType { { func( Index )... } };
                }

                // --- (for sort) ---

                // LMB
                constexpr std::size_t LMB_mask( std::size_t n, std::size_t mask ) { return n&mask ? n&mask : n; }
                template<std::size_t Bits = sizeof( std::size_t ) * 8 / 2>
                constexpr std::size_t LMB_loop( std::size_t n, std::size_t mul ) {
                        return Bits == 0 ? n : LMB_loop<Bits / 2>( LMB_mask( n, ~( ( ( std::size_t( 1 ) << Bits ) - 1 ) * mul ) ), mul * ( ( std::size_t( 1 ) << Bits ) + 1 ) );
                }
                constexpr std::size_t LMB( std::size_t n ) { return LMB_loop( n, 1 ); }

                // bitonic_sort
                template<class T, std::size_t N, class Comparator>
                constexpr T sorter( const T( &arr )[N], std::size_t i, std::size_t j, const Comparator& comp ) {
                        return j >= N ? arr[i] :
                               i < j  ? comp( arr[i], arr[j] ) ? arr[i] : arr[j]
                                      : comp( arr[i], arr[j] ) ? arr[j] : arr[i];
                }
                template<class T, std::size_t N, class Comparator, lfl::index_t... Index>
                constexpr lfl::s_array<T, N> bitonic_sort_sort( const T( &arr )[N], std::size_t xor_val, const Comparator& comp, lfl::index_tuple<Index...> ) {
                        return { lfl::detail::sorter( arr, Index, Index^xor_val, comp )... };
                }
                template<class T, std::size_t N, class Comparator>
                constexpr lfl::s_array<T, N> bitonic_sort_loop2( const T( &arr )[N], std::size_t sub_N, std::size_t subsub_N, const Comparator& comp ) {
                        return subsub_N == 1 ? lfl::detail::bitonic_sort_sort( arr, sub_N * 2 - 1, comp, lfl::make_index_tuple<N>{} )
                                             : lfl::detail::bitonic_sort_sort( (const T(&)[N])lfl::detail::bitonic_sort_loop2( arr, sub_N, subsub_N / 2, comp ), sub_N / subsub_N, comp, lfl::make_index_tuple<N>{} );
                }
                template<class T, std::size_t N, class Comparator>
                constexpr lfl::s_array<T, N> bitonic_sort_loop1( const T( &arr )[N], std::size_t sub_N, const Comparator& comp ) {
                        return sub_N == 1 ? lfl::detail::bitonic_sort_loop2( arr, 1, 1, comp )
                                          : lfl::detail::bitonic_sort_loop2( (const T(&)[N])lfl::detail::bitonic_sort_loop1( arr, sub_N / 2, comp ), sub_N, sub_N, comp );
                }

        }

        // generate
        template<std::size_t N, class Func,
                 class RetType = lfl::s_array<decltype( std::declval<Func>()( std::declval<lfl::index_t>() ) ), N>>
        constexpr RetType generate( const Func& func ) {
                return lfl::detail::generate_impl<N>( func, lfl::make_index_tuple<N>{} );
        }

        // sort
        template<class T, std::size_t N, class Comparator>
        constexpr lfl::s_array<T, N> sort( const lfl::s_array<T, N>& arr, const Comparator& comp ) {                return lfl::detail::bitonic_sort_loop1( (const T(&)[N])arr, lfl::detail::LMB( N - 1 ), comp );
        }
} // lfl

#endif // LFL_LFL_HPP
```
