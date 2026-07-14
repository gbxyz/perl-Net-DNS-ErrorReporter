# SYNOPSIS

    use Net::DNS;
    use Net::DNS::ErrorReporter;

    my $resolver = Net::DNS::Resolver->new(
        nameservers => \@some_authority_servers,
    );

    my $answer = $resolver->send('example.org', 'HINFO');

    if (!do_some_checks_on($answer)) {
        my $reporter = Net::DNS::ErrorReporter->new;

        $reporter->report(
            packet  => $answer,
            error   => Net::DNS::ErrorReporter::SOME_ERROR_CODE,
        );
    }

# DESCRIPTION

[RFC 9567: DNS Error Reporting](https://www.rfc-editor.org/info/rfc9567/)
describes a method for a resolver to automatically signal an error to a
monitoring agent specified by the authoritative server. This shortens the time
between an issue affecting a zone occurring and the operator of that zone being
aware of it.

DNS Error Reporting is used by
[RFC 9859: Generalized DNS Notifications](https://www.rfc-editor.org/info/rfc9859/).
When a child operator sends a `NOTIFY` message to the parent operator's endpoint,
they set the "agent domain" field in the "report channel" EDNS0 record, which
the parent can use to notify them of any issues found during CDS/CDNSKEY/CSYNC
scanning (see [RFC 10026](https://www.rfc-editor.org/info/rfc10026/)).

# USAGE

## INSTANTIATION

    $reporter = Net::DNS::ErrorReporter->new;

This instantiates a new reporter object.

## SENDING ERROR REPORTS

    $result = $reporter->report(
        packet  => $packet,
        error   => Net::DNS::ErrorReporter::SOME_ERROR_CODE,
    );

This sends an error report for the DNS message in `$packet`, which must be a
[Net::DNS::Packet](https://metacpan.org/pod/Net%3A%3ADNS%3A%3APacket) containing a query response. If `$packet` is invalid in
some way, does not contain the required EDNS0 option, or the report query failed,
`report()` will return `undef`. Otherwise, it will return a [Net::DNS::Packet](https://metacpan.org/pod/Net%3A%3ADNS%3A%3APacket)
containing the response (from the resolver) to the error report query.

The value of the `error` parameter must be an integer taken from the
[Extended DNS Error Codes](https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#extended-dns-error-codes)
IANA registry. For readability, [Net::DNS::ErrorReporter](https://metacpan.org/pod/Net%3A%3ADNS%3A%3AErrorReporter) provides the following
constants. The semantics of each code can be found by following the references
in the aforementioned IANA registry.

- `Net::DNS::ErrorReporter::OTHER_ERROR` (0)
- `Net::DNS::ErrorReporter::UNSUPPORTED_DNSKEY_ALGORITHM` (1)
- `Net::DNS::ErrorReporter::UNSUPPORTED_DS_DIGEST_TYPE` (2)
- `Net::DNS::ErrorReporter::STALE_ANSWER` (3)
- `Net::DNS::ErrorReporter::FORGED_ANSWER` (4)
- `Net::DNS::ErrorReporter::DNSSEC_INDETERMINATE` (5)
- `Net::DNS::ErrorReporter::DNSSEC_BOGUS` (6)
- `Net::DNS::ErrorReporter::SIGNATURE_EXPIRED` (7)
- `Net::DNS::ErrorReporter::SIGNATURE_NOT_YET_VALID` (8)
- `Net::DNS::ErrorReporter::DNSKEY_MISSING` (9)
- `Net::DNS::ErrorReporter::RRSIGS_MISSING` (10)
- `Net::DNS::ErrorReporter::NO_ZONE_KEY_BIT_SET` (11)
- `Net::DNS::ErrorReporter::NSEC_MISSING` (12)
- `Net::DNS::ErrorReporter::CACHED_ERROR` (13)
- `Net::DNS::ErrorReporter::NOT_READY` (14)
- `Net::DNS::ErrorReporter::BLOCKED` (15)
- `Net::DNS::ErrorReporter::CENSORED` (16)
- `Net::DNS::ErrorReporter::FILTERED` (17)
- `Net::DNS::ErrorReporter::PROHIBITED` (18)
- `Net::DNS::ErrorReporter::STALE_NXDOMAIN_ANSWER` (19)
- `Net::DNS::ErrorReporter::NOT_AUTHORITATIVE` (20)
- `Net::DNS::ErrorReporter::NOT_SUPPORTED` (21)
- `Net::DNS::ErrorReporter::NO_REACHABLE_AUTHORITY` (22)
- `Net::DNS::ErrorReporter::NETWORK_ERROR` (23)
- `Net::DNS::ErrorReporter::INVALID_DATA` (24)
- `Net::DNS::ErrorReporter::SIGNATURE_EXPIRED_BEFORE_VALID` (25)
- `Net::DNS::ErrorReporter::TOO_EARLY` (26)
- `Net::DNS::ErrorReporter::UNSUPPORTED_NSEC3_ITERATIONS_VALUE` (27)
- `Net::DNS::ErrorReporter::UNABLE_TO_CONFORM_TO_POLICY` (28)
- `Net::DNS::ErrorReporter::SYNTHESIZED` (29)
- `Net::DNS::ErrorReporter::INVALID_QUERY_TYPE` (30)
- `Net::DNS::ErrorReporter::RATE_LIMITED` (31)
- `Net::DNS::ErrorReporter::OVER_QUOTA` (32)
- `Net::DNS::ErrorReporter::NEGATIVE_TRUST_ANCHOR` (33)
- `Net::DNS::ErrorReporter::NEW_DELEGATION_ONLY` (34)

## OTHER METHODS

    $resolver = $reporter->resolver;

This returns the reporter's internal resolver object, in case you need to tweak
any of its settings.
