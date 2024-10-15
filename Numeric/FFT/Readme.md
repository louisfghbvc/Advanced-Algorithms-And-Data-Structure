## FFT/NTT

### template

- 多項式的卷積, 從係數轉成value representation
- 使用FFT進行complex運算 Divide and Conquer達到O(nlogn)
- NTT對於mod值的FFT
- https://usaco.guide/adv/fft

### Note - FFT Killer
  - The "multiplication with arbitrary modulus" described in cp-algo requires long double to pass.

### inplace FFT
```cpp
using cd = complex<double>;
const double PI = acos(-1);

void fft(vector<cd> & a, bool invert) {
    int n = a.size();

    for (int i = 1, j = 0; i < n; i++) {
        int bit = n >> 1;
        for (; j & bit; bit >>= 1)
            j ^= bit;
        j ^= bit;

        if (i < j)
            swap(a[i], a[j]);
    }

    for (int len = 2; len <= n; len <<= 1) {
        double ang = 2 * PI / len * (invert ? -1 : 1);
        cd wlen(cos(ang), sin(ang));
        for (int i = 0; i < n; i += len) {
            cd w(1);
            for (int j = 0; j < len / 2; j++) {
                cd u = a[i+j], v = a[i+j+len/2] * w;
                a[i+j] = u + v;
                a[i+j+len/2] = u - v;
                w *= wlen;
            }
        }
    }

    if (invert) {
        for (cd & x : a)
            x /= n;
    }
}

vector<int> multiply(vector<int> const& a, vector<int> const& b) {
    vector<cd> fa(a.begin(), a.end()), fb(b.begin(), b.end());
    int n = 1;
    while (n < a.size() + b.size()) 
        n <<= 1;
    fa.resize(n);
    fb.resize(n);

    fft(fa, false);
    fft(fb, false);
    for (int i = 0; i < n; i++)
        fa[i] *= fb[i];
    fft(fa, true);

    vector<int> result(n);
    for (int i = 0; i < n; i++)
        result[i] = round(fa[i].real());
    return result;
}        
```

### FFT WITH MOD
```cpp=
/**
 * Author: Ludo Pulles, chilli, Simon Lindholm
 * Date: 2019-01-09
 * License: CC0
 * Source: http://neerc.ifmo.ru/trains/toulouse/2017/fft2.pdf (do read, it's
 excellent) Accuracy bound from http://www.daemonology.net/papers/fft.pdf
 * Description: fft(a) computes $\hat f(k) = \sum_x a[x] \exp(2\pi i \cdot k x /
 N)$ for all $k$. N must be a power of 2. Useful for convolution:
   \texttt{conv(a, b) = c}, where $c[x] = \sum a[i]b[x-i]$.
   For convolution of complex numbers or more than two vectors: FFT, multiply
   pointwise, divide by n, reverse(start+1, end), FFT back.
   Rounding is safe if $(\sum a_i^2 + \sum b_i^2)\log_2{N} < 9\cdot10^{14}$
   (in practice $10^{16}$; higher for random inputs).
   Otherwise, use NTT/FFTMod.
 * Time: O(N \log N) with $N = |A|+|B|$ ($\tilde 1s$ for $N=2^{22}$)
 * Status: somewhat tested
 * Details: An in-depth examination of precision for both FFT and FFTMod can be
 found
 * here
 (https://github.com/simonlindholm/fft-precision/blob/master/fft-precision.md)
 */

typedef complex<double> C;
void fft(vector<C> &a) {
    int n = sz(a), L = 31 - __builtin_clz(n);
    static vector<complex<long double>> R(2, 1);
    static vector<C> rt(2, 1); // (^ 10% faster if double)
    for (static int k = 2; k < n; k *= 2) {
        R.resize(n);
        rt.resize(n);
        auto x = polar(1.0L, acos(-1.0L) / k);
        for (int i = k; i < 2 * k; ++i)
            rt[i] = R[i] = i & 1 ? R[i / 2] * x : R[i / 2];
    }
    vector<int> rev(n);
    for (int i = 0; i < n; ++i)
        rev[i] = (rev[i / 2] | (i & 1) << L) / 2;
    for (int i = 0; i < n; ++i)
        if (i < rev[i])
            swap(a[i], a[rev[i]]);
    for (int k = 1; k < n; k *= 2)
        for (int i = 0; i < n; i += 2 * k)
            for (int j = 0; j < k; ++j) {
                // C z = rt[j+k] * a[i+j+k]; // (25% faster if hand-rolled)  ///
                // include-line
                auto x = (double *)&rt[j + k],
                     y = (double *)&a[i + j + k]; /// exclude-line
                C z(x[0] * y[0] - x[1] * y[1],
                    x[0] * y[1] + x[1] * y[0]); /// exclude-line
                a[i + j + k] = a[i + j] - z;
                a[i + j] += z;
            }
}

typedef vector<LL> vl;
template <int M>
vl convMod(const vl &a, const vl &b) {
    if (a.empty() || b.empty())
        return {};
    vl res(sz(a) + sz(b) - 1);
    int B = 32 - __builtin_clz(sz(res)), n = 1 << B, cut = int(sqrt(M));
    vector<C> L(n), R(n), outs(n), outl(n);
    for (int i = 0; i < sz(a); ++i)
        L[i] = C((int)a[i] / cut, (int)a[i] % cut);
    for (int i = 0; i < sz(b); ++i)
        R[i] = C((int)b[i] / cut, (int)b[i] % cut);
    fft(L), fft(R);
    for (int i = 0; i < n; ++i) {
        int j = -i & (n - 1);
        outl[j] = (L[i] + conj(L[j])) * R[i] / (2.0 * n);
        outs[j] = (L[i] - conj(L[j])) * R[i] / (2.0 * n) / 1i;
    }
    fft(outl), fft(outs);
    for (int i = 0; i < sz(res); ++i) {
        LL av = LL(real(outl[i]) + .5), cv = LL(imag(outs[i]) + .5);
        LL bv = LL(imag(outl[i]) + .5) + LL(real(outs[i]) + .5);
        res[i] = ((av % M * cut + bv) % M * cut + cv) % M;
    }
    return res;
}
```

### NTT


### 例題

https://codeforces.com/contest/632/problem/E
